# mount: wrong fs type, bad option, bad superblock on /dev/sdX, missing codepage or helper program

> mount is a catch-all message. It means the kernel or a helper program could not mount the device with the given type and options. The kernel log usually names the real reason.

Example:

```
$ sudo mount /dev/sdb1 /mnt/data
mount: /mnt/data: wrong fs type, bad option, bad superblock on /dev/sdb1, missing codepage or helper program, or other error.
```

## Common causes

- The helper for this filesystem is missing: `nfs-common` for NFS, `cifs-utils` for SMB, `ntfs-3g` or `exfatprogs` for NTFS and exFAT.
- The device has no filesystem yet, or a different type than you gave, or you mounted the disk instead of its partition.
- A bad option or a typo in `/etc/fstab`.
- The filesystem is damaged or was not shut down cleanly.
- The device name changed after a reboot, so the line points to the wrong disk.

## How to check

What does the device contain?:

```bash
lsblk -f
```

Why did the kernel refuse? Read the last lines:

```bash
journalctl -k -n 20 --no-pager
```

Type and UUID of the partition:

```bash
sudo blkid /dev/sdb1
```

Does fstab look valid?:

```bash
findmnt --verify
```

## Fix

1. Read the kernel log first, it often says the exact reason.
2. Install the missing helper for the filesystem and mount again, for example `sudo apt install nfs-common`, `cifs-utils` or `ntfs-3g`.
3. Mount the right device with the right type: `sudo mount -t ext4 /dev/sdb1 /mnt/data`. In fstab use `UUID=` instead of `/dev/sdX`, then test with `sudo mount -a`.
4. If it looks damaged, check it read-only first while unmounted: `sudo fsck -n /dev/sdb1`. Repair with `fsck` only after a backup. A repair can change or remove data.
5. If `blkid` shows nothing, the disk has no filesystem. Creating one with `mkfs` erases everything on it. Do it only on a disk you are sure is empty.

## FAQ

**Is my data gone?**

Not necessarily. The message is generic. Do not run mkfs or a repair before you know the cause and have a copy, for example an image made with `dd` on another disk.

**Why does it fail only at boot?**

A wrong line in `/etc/fstab` or a disk that is not ready. Add `nofail` for non-essential disks so the boot does not stop.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mount-wrong-fs-type-bad-option-bad-superblock/](https://daniele-michelin.com/en/errors/mount-wrong-fs-type-bad-option-bad-superblock/)**
