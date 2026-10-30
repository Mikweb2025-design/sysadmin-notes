# PHP Fatal error: Uncaught Error: Call to undefined function mysqli_connect()

> The PHP that runs your page has no `mysqli` extension loaded. The code is fine, the module is missing for this PHP version or this SAPI (CLI and PHP-FPM have separate configuration).

Example:

```
PHP Fatal error:  Uncaught Error: Call to undefined function mysqli_connect() in /var/www/example.com/db.php:4
Stack trace:
#0 /var/www/example.com/index.php(2): require()
```

## Common causes

- The package with the MySQL extensions (`php-mysql`) is not installed.
- It was installed for another PHP version than the one that serves the site, for example after an upgrade with several versions side by side.
- The extension is installed but disabled in the `conf.d` of the FPM configuration.
- PHP-FPM was not restarted after the installation, so the running workers still lack the module.
- In a container, the extension was never built into the image.

## How to check

Does the CLI PHP know mysqli?:

```bash
php -m | grep -i mysql
```

Which PHP-MySQL packages are installed?:

```bash
dpkg -l | grep -E '^ii +php[0-9.]*-mysql'
```

Does each installed PHP-FPM binary list mysqli? (this is what the site uses):

```bash
for f in /usr/sbin/php-fpm*; do echo "$f"; $f -m | grep -i mysqli; done
```

Is it enabled in the FPM configuration?:

```bash
ls -l /etc/php/*/fpm/conf.d/ | grep -i mysql
```

## Fix

1. Install the extension for the version in use: `apt install php-mysql`, or `php8.x-mysql` with the exact version name of your FPM.
2. Enable it if needed: `phpenmod mysqli`.
3. Restart the PHP-FPM service of that version so the workers load the module.
4. Check again with the PHP-FPM binary of that version: `php-fpm8.x -m | grep mysqli`.
5. In a Docker image add `docker-php-ext-install mysqli` to the Dockerfile and rebuild.

## FAQ

**Why does it work on the command line but not in the browser?**

The CLI and PHP-FPM use different configuration and can even be different PHP versions. Test with the PHP-FPM binary, not only with `php -m`.

**Is the same cause behind Class mysqli not found?**

Yes. Both messages come from the missing `mysqli` extension. The same applies to PDO: `could not find driver` means `pdo_mysql` is missing, which the same package provides.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-call-to-undefined-function-mysqli-connect/](https://daniele-michelin.com/en/errors/php-call-to-undefined-function-mysqli-connect/)**
