# No space left on device although df shows free space

> The kernel returns ENOSPC because a filesystem resource is exhausted – not necessarily disk space, often the inodes or space held by deleted files that are still open.

Example:

```
cp: cannot create regular file '/var/www/nextcloud/data/upload.tmp': No space left on device
```

## Common causes

- All inodes used up by millions of small files (PHP sessions, cache, mail queue, Maildir).
- Deleted but still open files (e.g. logs) keep using space until the process closes them.
- Blocks reserved for root (5% by default on ext4): regular users already see the disk as full.
- A different filesystem is full, e.g. `/tmp` on tmpfs or a Docker volume.
- A user or group quota is reached.

## How to check

Space per filesystem:

```bash
df -h
```

Inode usage per filesystem:

```bash
df -i
```

Find deleted files that are still open:

```bash
sudo lsof +L1
```

Find directories with the most files:

```bash
sudo du --inodes -x /var 2>/dev/null | sort -n | tail -n 15
```

Show reserved blocks (ext4):

```bash
sudo tune2fs -l /dev/sda1 | grep -i 'reserved block count'
```

## Fix

1. Delete masses of small files in a targeted way, e.g. old sessions: `find /var/lib/php/sessions -type f -mmin +1440 -delete`.
2. Restart the process holding a deleted file, or truncate it via `: > /proc//fd/`.
3. Configure logrotate properly (`postrotate` with reload or `copytruncate`) so open logs are not deleted.
4. On pure data disks, lower the reservation: `tune2fs -m 1 /dev/sdb1`.
5. Automate cleanup via cron and monitor inode usage.

## FAQ

**Can I increase the inode count afterwards?**

Not on ext4 without reformatting. XFS and Btrfs allocate inodes dynamically. Long term, remove the cause of all the small files.

**Why is no space freed after deleting the big log file?**

The process still has the file open. The kernel only frees the space after the service is restarted or reloaded – `lsof +L1` shows these cases.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/linux-no-space-left-on-device/](https://daniele-michelin.com/en/errors/linux-no-space-left-on-device/)**
