# Nextcloud: Your data directory is invalid (.ocdata missing)

> Nextcloud refuses to start because it does not find the marker file `.ocdata` in the data directory. This protects you: Nextcloud will not write into a folder that might be an empty, unmounted mount point.

Example:

```
Your data directory is invalid
Ensure there is a file called ".ocdata" in the root of the data directory.
```

## Common causes

- The data volume (second disk, NFS, SMB, bind mount) is not mounted after a reboot, so the path exists but is empty.
- `datadirectory` in `config.php` points to a different path than the real one, for example after a move or a migration.
- The hidden file `.ocdata` was not copied, because the copy skipped dot files or only some folders were restored from a backup.
- The web server user cannot read the directory, for example after wrong `chown` or a changed mount option.

## How to check

Which path does Nextcloud use?:

```bash
grep datadirectory /var/www/nextcloud/config/config.php
```

Is the marker file there?:

```bash
ls -la /srv/nextcloud-data/.ocdata
```

Is the volume really mounted?:

```bash
findmnt /srv/nextcloud-data
```

Owner and mode of the directory:

```bash
stat -c '%U:%G %a %n' /srv/nextcloud-data
```

## Fix

1. Mount the volume first and check that your user folders and `nextcloud.log` are visible in the data directory. Make the mount start before the web server, for example with `RequiresMountsFor=/srv/nextcloud-data` in a systemd drop-in.
2. If the path in `config.php` is wrong, set the real one in `datadirectory` and nowhere else.
3. Only when the folder is the real data directory and the file is missing, recreate it: `sudo -u www-data touch /srv/nextcloud-data/.ocdata`. Never do this on an empty mount point.
4. Fix the owner: `chown -R www-data:www-data /srv/nextcloud-data` and mode `0750` on the directory.
5. Reload the page. For future copies use `rsync -a`, which keeps dot files.

## FAQ

**Can I just create .ocdata to make the error go away?**

Only if you are sure the directory holds your real data. On an unmounted empty folder it would make Nextcloud start with a new, empty instance, and the first uploads would land on the wrong disk.

**Why does it happen after every reboot?**

Usually the mount comes up later than the web server or fails silently. Check the mount unit or the `fstab` line and add a dependency so the web server waits for it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-your-data-directory-is-invalid-ocdata/](https://daniele-michelin.com/en/errors/nextcloud-your-data-directory-is-invalid-ocdata/)**
