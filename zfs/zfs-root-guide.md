# Gentoo ZFS root guide

Personal Install Guide

## Partition Setup

This guide uses ZFSBootMenu as the bootloader installed onto a usb drive. This allows ZFS to have the entire root disk to itself. For the usb drive either use an internal adapter <https://www.amazon.com/gp/product/B06Y5C7DKH?psc=1>, or just plug the usb drive into a port on the back of the motherboard.

EFI USB setup

```bash
mklabel gpt
unit mib

mkpart primary 1 -1    # EFI
name 1 esp
set 1 boot on
```

## Formating Partitions

### Formating the EFI partition

```bash
mkfs.fat -F32 /dev/<efi part> # use the esp partition from the previous step
```

### Creating the ZFS root pool and datasets

```bash
mkdir /mnt/gentoo # install location
```

ZFS single disk command

```bash
zpool create -f -o ashift=12 -o autotrim=on -O acltype=posixacl -O xattr=sa -O dnodesize=auto -O utf8only=on -O normalization=formD -o cachefile= -O atime=off -O relatime=on -O compression=zstd -m none -R /mnt/gentoo zroot /dev/nvme0n1
```

ZFS mirror disks command

```bash
zpool create -f -o ashift=12 -o autotrim=on -O acltype=posixacl -O xattr=sa -O dnodesize=auto -O utf8only=on -O normalization=formD -o cachefile= -O atime=off -O relatime=on -O compression=zstd -m none -R /mnt/gentoo zroot mirror /dev/nvme0n1 /dev/nvme1n1
```

## ZFS Datasets

ZFSBootMenu requires the contents of /boot to be in the same pool that is will be booting. Do not use a separate dataset or pool.

```bash
# Top level datasets
zfs create zroot/ROOT
zfs create zroot/HOME

# system
zfs create -o mountpoint=/ zroot/ROOT/gentoo

# /var - external state that should not get rolled back by snapshots
zfs create zroot/ROOT/gentoo/var -o canmount=off
zfs create zroot/ROOT/gentoo/var/db -o canmount=off
zfs create zroot/ROOT/gentoo/var/db/repos
# zfs create zroot/ROOT/gentoo/var/db/pkg # Should be keep in sync with root pool
zfs create zroot/ROOT/gentoo/var/spool
zfs create zroot/ROOT/gentoo/var/lib
zfs create zroot/ROOT/gentoo/var/log
zfs create zroot/ROOT/gentoo/var/distfiles

# /usr/local - optional
zfs create zroot/ROOT/gentoo/usr -o canmount=no
zfs create zroot/ROOT/gentoo/usr/local

# /home/users
zfs create -o mountpoint=/home/brandon zroot/HOME/brandon

# /root
zfs create -o mountpoint=/root zroot/HOME/root
```

## Gentoo Install

### ZFSBootMenu kernel requirments

Linux kernel should be built with `CONFIG_RELOCATABLE`, this fixed kexec failing to load kernels with `locate_hole failed`.

### New install

Now just install gentoo.

<https://wiki.gentoo.org/wiki/Handbook:AMD64>

### Copy an existing ZFS root

Copy an existing root pool from a snapshot. Will have to export the new pool to use send/recv

```bash
zpool export zroot
zfs send rpool/ROOT/gentoo@snapshot | zfs recv -F zroot/ROOT/gentoo
zpool import zroot -R /mnt/gentoo
```

Or using a snapshot file.

```bash
# Existing system export
zfs snapshot zroot/ROOT/gentoo@export
# Make sure to use somewhere with enough disk space
zfs send zroot/ROOT/gentoo@export | zstd > system.snapshot.zst

# Then either move system.snapshot.zst to the new system or mount an NFS share with the file.

# New system install
zpool export zroot
zstd -T0 -dc system.snapshot.zst > zfs recv -F zroot/ROOT/gentoo
zpool import zroot -R /mnt/gentoo
```

### Gentoo ZFS kernel builtin

To build Linux with zfs support builtin and generate an initramfs use the `kernel-build` script provided in <https://github.com/thermitegod/shell-scripts>

## Gentoo ZFS Bootloader Install

Add /boot/efi to /etc/fstab using `blkid`

```/etc/fstab
PARTUUID=<ID>  /boot/efi  vfat  defaults  0 2
```

To check existing and/or delete boot entries.

```bash
# list all entries
efibootmgr

# delete an entry
efibootmgr -b <num> -B <num>
```

Setup ZFSBootMenu

```bash
mkdir -p /boot/efi
mount /boot/efi
mkdir /boot/efi/EFI/zbm

zpool set bootfs=zroot/ROOT/gentoo zroot

wget https://get.zfsbootmenu.org/latest.EFI -O /boot/efi/EFI/zbm/zfsbootmenu.EFI

# for the disk use the device from /dev/disk/by-id/<disk>, not the partition, i.e. <disk>-part1
efibootmgr --disk <disk> --part 1 --create --label "ZFSBootMenu" --loader '\EFI\zbm\zfsbootmenu.EFI' --unicode "spl.spl_hostid=$(hostid) zbm.timeout=10 zbm.prefer=zroot zbm.import_policy=force" --verbose

# Set kernel parameters
zfs set org.zfsbootmenu:commandline="rw consoleblank=0 spl.spl_hostid=$(hostid)" zroot/ROOT
```

Next run `efibootmgr` as a sanity check to ensure that the new boot entries have been made.

Example of a good output for ZFSBootMenu.

```bash
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0000
Boot0000* ZFSBootMenu	HD(1,GPT,8952486c-ed08-4af4-858f-cd24510c3dec,0x800,0x1cea000)/File(\EFI\zbm\zfsbootmenu.EFI)
```

### After Gentoo install

required init services for ZFS root

#### Openrc

```bash
rc-update add zfs-import boot
rc-update add zfs-mount boot
```

#### Systemd

```bash
systemctl enable zfs.target

systemctl enable zfs-import-cache
systemctl enable zfs-mount
systemctl enable zfs-import.target
```

### Initramfs fails to mount pools

If the initramfs fails to mount the root pool on the first boot after installing through a livecd, enter the emergency shell in the initramfs and run the following.

```bash
zpool import -f zroot -R /sysroot
exit
```

The system should then continue booting.

## External Links

<https://wiki.gentoo.org/wiki/User:Fearedbliss/Installing_Gentoo_Linux_On_ZFS>

<https://openzfs.github.io/openzfs-docs/Getting%20Started/Arch%20Linux/Root%20on%20ZFS/2-system-installation.html>

<https://docs.zfsbootmenu.org/>

<https://docs.zfsbootmenu.org/en/v2.2.x/guides/general/portable.html>
