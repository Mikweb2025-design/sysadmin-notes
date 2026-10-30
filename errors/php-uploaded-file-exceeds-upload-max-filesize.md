# PHP: The uploaded file exceeds the upload_max_filesize directive in php.ini

> PHP rejected the upload because the file is larger than `upload_max_filesize`, or the whole request is larger than `post_max_size`. Several limits apply in a chain, from the web server to PHP.

Example:

```
The uploaded file exceeds the upload_max_filesize directive in php.ini
PHP Warning:  POST Content-Length of 52428900 bytes exceeds the limit of 8388608 bytes in Unknown on line 0
```

## Common causes

- `upload_max_filesize` in the `php.ini` of PHP-FPM is smaller than the file.
- `post_max_size` is smaller than the upload plus other form data (then `$_FILES` is silently empty).
- nginx `client_max_body_size` is smaller and answers 413 before PHP sees the file.
- The setting was changed in the CLI `php.ini` instead of the FPM one.
- A pool or `.user.ini` overrides the global value, or the upload is so slow that `max_input_time` or `max_execution_time` ends it.

## How to check

Effective values in the FPM php.ini:

```bash
grep -E '^(upload_max_filesize|post_max_size|memory_limit|max_input_time|max_execution_time) *=' /etc/php/*/fpm/php.ini
```

Overrides in conf.d and pools:

```bash
grep -rEn 'upload_max_filesize|post_max_size' /etc/php/*/fpm/conf.d/ /etc/php/*/fpm/pool.d/
```

The nginx limit:

```bash
nginx -T 2>/dev/null | grep -n 'client_max_body_size'
```

What does the web server return for a large body? (look at the status line):

```bash
curl -s -o /dev/null -w '%{http_code}\n' -X POST --data-binary @/etc/services https://example.com/upload.php
```

## Fix

1. Create a drop-in `/etc/php//fpm/conf.d/99-upload.ini` with `upload_max_filesize = 64M` and `post_max_size = 72M`. The second value must be larger than the first.
2. Make sure `memory_limit` is not smaller than `post_max_size` if the application loads the file into memory.
3. In nginx set `client_max_body_size 72M;` (in `http`, `server` or the upload `location`).
4. Restart the PHP-FPM service of your version and reload nginx: `nginx -t && systemctl reload nginx`.
5. For slow connections also raise `max_input_time` and `max_execution_time`.

## FAQ

**Why must post_max_size be bigger than upload_max_filesize?**

The POST request contains the file plus form fields and encoding overhead. If it exceeds `post_max_size`, PHP drops the whole request body.

**Does .htaccess help on nginx?**

No. nginx ignores `.htaccess`. Use the php.ini drop-in, or `php_admin_value` in the pool file.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-uploaded-file-exceeds-upload-max-filesize/](https://daniele-michelin.com/en/errors/php-uploaded-file-exceeds-upload-max-filesize/)**
