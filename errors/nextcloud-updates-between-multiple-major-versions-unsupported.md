# Nextcloud: Updates between multiple major versions and downgrades are unsupported

> Nextcloud only upgrades from one major version to the next. The message appears when you skip a major, or when the code you installed is older than the version recorded in the database. Never force it by editing the version number.

Example:

```
Nextcloud or one of the apps require upgrade - only a limited number of operations are allowed
Updates between multiple major versions and downgrades are unsupported.
Update failed
```

## Common causes

- You tried to jump over a major release, for example from version N straight to N+2.
- The code is older than the data: a file backup or an old Docker image tag was restored, but the database and `config.php` belong to a newer version.
- The wrong code folder is live, for example a symlink or web root still pointing to an old copy after an update.
- An update was interrupted halfway, so code and database now disagree.

## How to check

Version of the code on disk:

```bash
grep OC_VersionString /var/www/nextcloud/version.php
```

Version Nextcloud has recorded:

```bash
grep "'version'" /var/www/nextcloud/config/config.php
```

What does the CLI say?:

```bash
sudo -u www-data php occ status
```

Where does the web root point?:

```bash
readlink -f /var/www/nextcloud
```

## Fix

1. Compare the two versions. If the code is older than the recorded one, put the matching newer code back, or restore a database backup that was taken together with this code.
2. If you skipped a major, install the last release of the next major first, run `sudo -u www-data php occ upgrade`, and repeat one major at a time until you reach the target.
3. Before each step, make a backup of code, `config.php`, data and database, and disable apps that do not support the next version with `occ app:disable`.
4. Do not edit `'version'` in `config.php` to get past the check. That can corrupt the database schema.
5. After the last step, leave maintenance mode with `occ maintenance:mode --off` and check the admin overview.

## FAQ

**Can I downgrade Nextcloud?**

Not by swapping the code. A downgrade is only safe by restoring a full backup (code, config and database) from the same moment.

**How do I know which majors I must install?**

Check the release notes and upgrade path of the official documentation for your starting version, then go through each major in order, always taking the latest minor release of it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-updates-between-multiple-major-versions-unsupported/](https://daniele-michelin.com/en/errors/nextcloud-updates-between-multiple-major-versions-unsupported/)**
