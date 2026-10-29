# PHP Fatal error: Allowed memory size exhausted

> A PHP script used more memory than `memory_limit` allows, so PHP stopped it. The default of 128 MB is often too little for image processing, imports or large plugins.

Example:

```
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted (tried to allocate 20480 bytes) in /var/www/html/wp-includes/class-wp-image-editor-imagick.php on line 612
```

## Common causes

- `memory_limit` is too low for the task: resizing large images, a big import, Composer, a heavy plugin.
- A loop or a query loads far too much data into memory at once.
- A memory leak in a long-running script or worker.
- The limit was raised in the wrong `php.ini`: the CLI and PHP-FPM read different files.
- A hosting panel or pool setting overrides the global value.

## How to check

Which php.ini does the command line use, and what is the limit?:

```bash
php --ini; php -r 'echo ini_get("memory_limit"), PHP_EOL;'
```

Where is memory_limit set for FPM?:

```bash
grep -rn 'memory_limit' /etc/php/*/fpm/
```

Which line failed? Read the log:

```bash
grep -i 'memory size' /var/log/nginx/error.log /var/log/php8.3-fpm.log | tail -n 10
```

## Fix

1. Raise the limit in the file PHP-FPM uses, for example `memory_limit = 512M` in `/etc/php/8.3/fpm/php.ini`, then `systemctl reload php8.3-fpm`.
2. Per pool, set `php_admin_value[memory_limit] = 512M` in the pool file.
3. On a panel such as Plesk, change the value in the domain's PHP settings.
4. For Composer, run `COMPOSER_MEMORY_LIMIT=-1 composer install` once.
5. If the script keeps growing, fix the cause: process data in chunks, free large variables, avoid loading whole tables.

## FAQ

**Is memory_limit = -1 (unlimited) a good idea?**

Not for web requests. One broken request can then use all RAM and bring the server down. Use a fixed value, large enough for your biggest legitimate task.

**I changed php.ini but nothing happened.**

Most likely you edited the CLI file, or PHP-FPM was not reloaded. Check which file is loaded and reload the service.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-allowed-memory-size-exhausted/](https://daniele-michelin.com/en/errors/php-allowed-memory-size-exhausted/)**
