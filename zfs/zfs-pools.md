# Pool Creation Commands

Incase I ever need these.

## 2018-02-23 Torrents

```bash
zpool create -f -O atime=off -o ashift=12 -O checksum=skein -O compression=lz4 -m /mnt/torrents torrents mirror /dev/disk/by-id/wwn-0x5000cca255c01c56 /dev/disk/by-id/wwn-0x50014ee260e55ce2
```

## 2019-07-08 SSD Mirror

```bash
zpool create -o ashift=12 -O atime=off -O compression=lz4 -m none ssd-mirror mirror /dev/disk/by-id/wwn-0x5002538da007852e /dev/disk/by-id/wwn-0x5002538d417e53d6

zfs create -o mountpoint=/mnt/kvm ssd-mirror/KVM
zfs create -o mountpoint=/mnt/music ssd-mirror/MUSIC
```

## 2019-07-08 Storage Pool (2019)

```bash
zpool create -f -O normalization=formD -O atime=off -o ashift=12 -O checksum=skein -O compression=lz4 -m none storage raidz2 /dev/disk/by-id/wwn-0x7001770231855206400x /dev/disk/by-id/wwn-0x6667096484546236416x /dev/disk/by-id/wwn-0x6674133358964002816x /dev/disk/by-id/wwn-0x15338777562753159168x /dev/disk/by-id/wwn-0x1668963248178745345x raidz2 /dev/disk/by-id/wwn-0x1153208487055347713x /dev/disk/by-id/wwn-0x6995859257344282624x /dev/disk/by-id/wwn-0x5337127219588386816x /dev/disk/by-id/wwn-0x6993888932507308032x /dev/disk/by-id/wwn-0x13633246000157446145x

zfs create -o mountpoint=/mnt/anime storage/anime
zfs create -o mountpoint=/mnt/data storage/data
```

## 2023-08-11 Storage Pool (2023)

```bash
zpool create -f -o ashift=12 -O acltype=posixacl -O xattr=sa -O dnodesize=auto -O utf8only=on -O normalization=formD -O atime=off -O relatime=on -O compression=zstd -m none working raidz2 /dev/disk/by-id/wwn-0x5000c5007532961e /dev/disk/by-id/wwn-0x5000c5007a6b963c /dev/disk/by-id/wwn-0x5000c5007abc65c8 /dev/disk/by-id/wwn-0x5000c5007a55a62f /dev/disk/by-id/wwn-0x5000c500753cb36e /dev/disk/by-id/wwn-0x5000c5007ac537ce /dev/disk/by-id/wwn-0x5000c500e48f98cb /dev/disk/by-id/wwn-0x5000c500e5d4629d /dev/disk/by-id/wwn-0x5000c500e48f97ce /dev/disk/by-id/wwn-0x5000c500e49ac80d

zfs create -o mountpoint=/mnt/new_anime working/anime
zfs create -o mountpoint=/mnt/new_data working/data

# export the old storage pool after copying files
zpool export storage

# rename the pool
zpool export working
zpool import working storage

# update mountpoints
zfs set mountpoint=/mnt/anime storage/anime
zfs set mountpoint=/mnt/data storage/data
```

## 2023-08-11 Cold Storage Pool (2023)

Reusing the storage pool (2019) drives for a backup pool in a raidz3. The wwn numbers are different than what I had saved from the 2019 creation, reason unknown.

```bash
zpool create -f -o ashift=12 -O acltype=posixacl -O xattr=sa -O dnodesize=auto -O utf8only=on -O normalization=formD -O atime=off -O relatime=on -O compression=zstd -m none icebox raidz3 /dev/disk/by-id/wwn-0x50014ee60501bd33 /dev/disk/by-id/wwn-0x50014ee605021001 /dev/disk/by-id/wwn-0x50014ee0593b1729 /dev/disk/by-id/wwn-0x5000cca248ed610f /dev/disk/by-id/wwn-0x5000cca248ed612b /dev/disk/by-id/wwn-0x5000cca248ecd4de /dev/disk/by-id/wwn-0x5000cca248ed5c86 /dev/disk/by-id/wwn-0x5000cca248ed4a11 /dev/disk/by-id/wwn-0x5000cca248ed5c9f /dev/disk/by-id/wwn-0x5000cca248ed6116

zfs create -o mountpoint=/mnt/icebox icebox/data
```
