# Nextcloud: “your data directory is readable by other users”

> On start Nextcloud checks whether other system users can read or list the data directory. If they can, Nextcloud refuses to run or shows the warning.

Example:

```
Your data directory is readable by other users. Please change the permissions to 0770 so that the directory cannot be listed by other users.
```

## Common causes

- The data directory has 0755 or 0775 instead of 0770, e.g. after a restore or `cp -r`.
- A new data directory was created as root and opened with `chmod 755` only.
- A CIFS/SMB or NFS mount presents fixed permissions that `chmod` cannot change.
- Docker volume with wrong UID/GID or permissions from the host.

## How to check

Read the data directory path from the config:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get datadirectory
```

Show permissions and owner:

```bash
stat -c '%a %U:%G %n' /var/www/nextcloud/data
```

Check filesystem and mount options:

```bash
findmnt -T /var/www/nextcloud/data -o TARGET,FSTYPE,OPTIONS
```

## Fix

1. Set permissions: `chmod 0770 /var/www/nextcloud/data` and owner `www-data:www-data`.
2. For CIFS mounts set permissions via mount options: `uid=www-data,gid=www-data,file_mode=0770,dir_mode=0770`.
3. For NFS set permissions on the server, not the client.
4. Only if the filesystem cannot do Unix permissions: `'check_data_directory_permissions' => false` in config.php – knowing the check is then gone.
5. Always check owner and permissions after restores.

## FAQ

**Is it enough to change only the top directory?**

For the warning, yes – the check looks at the data directory itself. Files below should still belong to the web server user.

**Why 0770 and not 0750?**

Both work. Nextcloud only requires that “others” have no access. 0750 is stricter if the group does not need to write.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-data-directory-readable-by-other-users/](https://daniele-michelin.com/en/errors/nextcloud-data-directory-readable-by-other-users/)**
