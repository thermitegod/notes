# Expanding ZFS root pool

This is after the new disks have been installed in the system and enough of the old zroot disks exist that the pool can still be imported.

```bash
  pool: zroot
 state: DEGRADED
status: One or more devices could not be used because the label is missing or
	invalid.  Sufficient replicas exist for the pool to continue
	functioning in a degraded state.
action: Replace the device using 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-4J
  scan: scrub repaired 0B in 00:04:21 with 0 errors on Mon Jun 12 00:10:03 2023
config:

	NAME                     STATE     READ WRITE CKSUM
	zroot                    DEGRADED     0     0     0
	  mirror-0               DEGRADED     0     0     0
	    9671111125011171242  UNAVAIL      0     0     0  was /dev/nvme1n1p3
	    nvme0n1p3            ONLINE       0     0     0

errors: No known data errors
```

Find the replacement disk in `/dev/disk/by-id`

```bash
lrwxrwxrwx 1 root root   13 2023-06-19 21:26:25 /dev/disk/by-id/nvme-eui.002538bc21a1504a -> ../../nvme2n1
lrwxrwxrwx 1 root root   13 2023-06-19 21:26:25 /dev/disk/by-id/nvme-eui.002538bc21a15378 -> ../../nvme1n1
lrwxrwxrwx 1 root root   13 2023-06-19 21:26:25 /dev/disk/by-id/nvme-eui.0025385b71b105ff -> ../../nvme0n1
```

```bash
zroot set autoexpand=on zroot
```

```bash
zpool replace zroot 9671111125011171242 "/dev/disk/by-id/nvme-eui.002538bc21a15378"
```

```bash
  pool: zroot
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
	continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Jun 19 21:41:49 2023
	355G scanned at 71.0G/s, 984K issued at 197K/s, 432G total
	0B resilvered, 0.00% done, no estimated completion time
config:

	NAME                             STATE     READ WRITE CKSUM
	zroot                            DEGRADED     0     0     0
	  mirror-0                       DEGRADED     0     0     0
	    replacing-0                  DEGRADED     0     0     0
	      9671111125011171242        UNAVAIL      0     0     0  was /dev/nvme1n1p3
	      nvme-eui.002538bc21a15378  ONLINE       0     0     0
	    nvme0n1p3                    ONLINE       0     0     0

errors: No known data errors
```

After the first resilvered has finished.

```bash
  pool: zroot
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
  scan: resilvered 433G in 00:05:23 with 0 errors on Mon Jun 19 21:47:12 2023
config:

	NAME                           STATE     READ WRITE CKSUM
	zroot                          ONLINE       0     0     0
	  mirror-0                     ONLINE       0     0     0
	    nvme-eui.002538bc21a15378  ONLINE       0     0     0
	    nvme0n1p3                  ONLINE       0     0     0

errors: No known data errors
```

Shutdown and replace the next disk.

```bash
zpool replace zroot nvme0n1p3 "/dev/disk/by-id/nvme-eui.002538bc21a1504a"
```

```bash
  pool: zroot
 state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
	continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Jun 19 21:53:44 2023
	354G scanned at 59.0G/s, 856K issued at 143K/s, 432G total
	0B resilvered, 0.00% done, no estimated completion time
config:

	NAME                             STATE     READ WRITE CKSUM
	zroot                            ONLINE       0     0     0
	  mirror-0                       ONLINE       0     0     0
	    nvme-eui.002538bc21a15378    ONLINE       0     0     0
	    replacing-1                  ONLINE       0     0     0
	      nvme0n1p3                  ONLINE       0     0     0
	      nvme-eui.002538bc21a1504a  ONLINE       0     0     0

errors: No known data errors
```

After the second resilvered has finished.

```bash
  pool: zroot
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
  scan: resilvered 433G in 00:06:22 with 0 errors on Mon Jun 19 22:00:06 2023
config:

	NAME                           STATE     READ WRITE CKSUM
	zroot                          ONLINE       0     0     0
	  mirror-0                     ONLINE       0     0     0
	    nvme-eui.002538bc21a15378  ONLINE       0     0     0
	    nvme-eui.002538bc21a1504a  ONLINE       0     0     0

errors: No known data errors
```
