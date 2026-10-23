# nginx + PHP-FPM: “Primary script unknown” / “File not found.”

> nginx passes the request to PHP-FPM, but PHP-FPM finds no file at the given path or may not read it. The browser just shows “File not found.”.

Example:

```
FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream, client: 198.51.100.7, request: "GET /index.php HTTP/1.1", upstream: "fastcgi://unix:/run/php/php8.3-fpm.sock:"
```

## Common causes

- `SCRIPT_FILENAME` is built incorrectly, e.g. without `$document_root`.
- `root` is set only inside `location /` and does not apply to `location ~ \.php$`.
- PHP-FPM runs in a container or chroot and sees a different path than nginx.
- The PHP-FPM user cannot read the file or a parent directory.
- The requested PHP file really does not exist and `try_files` is missing.

## How to check

Show affected requests in the log:

```bash
grep 'Primary script unknown' /var/log/nginx/error.log | tail -n 5
```

Show root, fastcgi_pass and SCRIPT_FILENAME in the active config:

```bash
nginx -T 2>/dev/null | grep -nE '^\s*(root|fastcgi_pass|fastcgi_param\s+SCRIPT_FILENAME|try_files) '
```

Show user and socket of the PHP-FPM pool:

```bash
grep -E '^(user|group|listen|chroot)' /etc/php/*/fpm/pool.d/www.conf
```

Check that the PHP-FPM user can read the file:

```bash
sudo -u www-data test -r /var/www/site/index.php && echo ok
```

## Fix

1. Set `root` at server level, not only in one location.
2. In the PHP block use `include fastcgi_params;` and `fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;`.
3. Add `try_files $fastcgi_script_name =404;` so missing scripts end in a clean 404.
4. With Docker mount the same path in the nginx and PHP containers.
5. Fix permissions for the pool user and run `nginx -t && systemctl reload nginx`.

## FAQ

**Why do I see “File not found.” instead of an nginx 404 page?**

The message comes from PHP-FPM, not nginx. nginx forwarded the request without checking first whether the file exists.

**Is this a security issue?**

Not by itself. But `try_files … =404` before `fastcgi_pass` also stops PHP-FPM from executing crafted paths like /upload/image.jpg/x.php.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-php-fpm-primary-script-unknown-file-not-found/](https://daniele-michelin.com/en/errors/nginx-php-fpm-primary-script-unknown-file-not-found/)**
