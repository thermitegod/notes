# ZFS Misc

## Temp mount for pool

```bash
zpool import -R /mnt <pool>
```

## Tunables

```bash
zfs set mountpoint=<dir> <pool>
zfs set atime=off <pool>
zfs set compression=lz4 <pool>
zfs set xattr=sa <pool>

zfs set autoexpand=on <pool>
zfs set autoreplace=on <pool>
```

## Rename a pool

```bash
zpool export <old pool>
zpool import <old pool> <new pool>
```
