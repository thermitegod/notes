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

## 2019-07-08 Storage

```bash
zpool create -f -O normalization=formD -O atime=off -o ashift=12 -O checksum=skein -O compression=lz4 -m none storage raidz2 /dev/disk/by-id/wwn-0x7001770231855206400x /dev/disk/by-id/wwn-0x6667096484546236416x /dev/disk/by-id/wwn-0x6674133358964002816x /dev/disk/by-id/wwn-0x15338777562753159168x /dev/disk/by-id/wwn-0x1668963248178745345x raidz2 /dev/disk/by-id/wwn-0x1153208487055347713x /dev/disk/by-id/wwn-0x6995859257344282624x /dev/disk/by-id/wwn-0x5337127219588386816x /dev/disk/by-id/wwn-0x6993888932507308032x /dev/disk/by-id/wwn-0x13633246000157446145x

zfs create -o mountpoint=/mnt/anime storage/anime
zfs create -o mountpoint=/mnt/data storage/data
```
