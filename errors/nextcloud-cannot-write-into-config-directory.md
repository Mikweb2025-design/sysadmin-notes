# Nextcloud: Cannot write into "config" directory!

> The web server user may not write to `config/`, so the installer, updates or settings cannot save `config.php`. Usually the owner or the permissions are wrong after a copy, restore or move.

Example:

```
Cannot write into "config" directory!
This can usually be fixed by giving the web server write access to the config directory. See https://docs.nextcloud.com/server/latest/admin_manual/installation/installation_wizard.html#setting-strong-directory-permissions
```

## Common causes

- Files were copied or restored as root, so the owner is not `www-data` (or `nginx`, `apache`).
- Permissions are too tight, for example mode 0700 for another user.
- SELinux or AppArmor blocks writing to the directory.
- In Docker, the mounted volume belongs to a different user than the one inside the container.
- `config_is_read_only` is set to `true` on purpose.

## How to check

Owner and permissions of the directory and the file:

```bash
ls -ld /var/www/nextcloud/config /var/www/nextcloud/config/config.php
```

Which user runs PHP?:

```bash
ps -eo user,comm | grep php-fpm | sort -u
```

Can that user write?:

```bash
sudo -u www-data test -w /var/www/nextcloud/config && echo writable
```

Is the config set to read-only?:

```bash
grep -n config_is_read_only /var/www/nextcloud/config/config.php
```

## Fix

1. Give the web server user the directory: `chown -R www-data:www-data /var/www/nextcloud/config`.
2. Set sensible permissions: `chmod 750 /var/www/nextcloud/config` and `chmod 640 /var/www/nextcloud/config/config.php`.
3. Only `config/`, `data/`, `apps/` and `themes/` need to be writable. Do not make the whole tree writable.
4. On SELinux, label the directory: `semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud/config(/.*)?'` and `restorecon -R /var/www/nextcloud/config`.
5. In Docker, fix the owner of the volume on the host, or set `config_is_read_only` back to `false` if it was not meant to be on.

## FAQ

**Is it safe to chown the whole Nextcloud folder?**

It works, but it is more than needed. Following the admin manual, the web server user only owns the four writable folders, and the code stays read-only for it.

**The error appears right after a restore from backup. Why?**

Backups often keep numeric user IDs. If the new system uses other IDs, the owner no longer matches. Set the owner again after the restore.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-cannot-write-into-config-directory/](https://daniele-michelin.com/en/errors/nextcloud-cannot-write-into-config-directory/)**
