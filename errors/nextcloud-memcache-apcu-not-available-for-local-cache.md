# Nextcloud: memcache \OC\Memcache\APCu not available for local cache

> Nextcloud is configured to use APCu as local cache, but the PHP instance currently running has APCu disabled. Almost always this only affects PHP CLI – occ and cron.php.

Example:

```
Memcache \OC\Memcache\APCu not available for local cache (Is the matching PHP module installed and enabled?)
```

## Common causes

- `apc.enable_cli` is not set to 1 in the CLI configuration (default is 0).
- `php-apcu` installed only for FPM, not for the CLI version.
- occ runs with a different PHP version than the web server, e.g. `php` points to 8.1 while FPM uses 8.3.
- config.php names APCu but the module was never installed.

## How to check

Check that the CLI loads the module:

```bash
php -m | grep -i apcu
```

Show apc.enable_cli and the CLI's loaded php.ini:

```bash
php -i | grep -E 'apc.enable_cli|Loaded Configuration File'
```

Show the configured local cache:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get memcache.local
```

Search existing entries in the CLI configuration:

```bash
grep -r 'apc.enable_cli' /etc/php/*/cli/
```

## Fix

1. Add `apc.enable_cli=1` to `/etc/php/8.3/mods-available/apcu.ini`.
2. Install the missing module: `apt install php8.3-apcu` – matching the PHP version in use.
3. Call occ and cron with the right version, e.g. `php8.3 occ` and `php8.3 -f cron.php`.
4. No APCu possible? Use `'memcache.local' => '\OC\Memcache\Redis'` instead.
5. Then check with `occ status` that the message is gone.

## FAQ

**Do I need to restart PHP-FPM?**

Not for the CLI – it reads its config on every call. Only if you reinstall the module for FPM do you need to restart PHP-FPM.

**Does this fix the cron job too?**

Often, yes. cron.php aborts with the same message, and Nextcloud then reports stale background jobs.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-memcache-apcu-not-available-for-local-cache/](https://daniele-michelin.com/en/errors/nextcloud-memcache-apcu-not-available-for-local-cache/)**
