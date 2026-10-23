# HTTP 403 Forbidden (nginx): directory index is forbidden

> nginx understands the request but refuses access. The exact reason is almost always in error.log – “directory index … is forbidden”, “Permission denied” or “access forbidden by rule”.

Example:

```
directory index of "/var/www/site/" is forbidden, client: 198.51.100.7, server: example.com, request: "GET / HTTP/1.1"
```

## Common causes

- The directory has no index file and `autoindex` is off (“directory index … is forbidden”).
- `index` does not list `index.php` although the app runs on PHP.
- The nginx user lacks execute (x) on a parent directory or read on the file (“13: Permission denied”).
- `deny` or `allow` rules block the client IP (“access forbidden by rule”).
- Wrong SELinux context, e.g. after `mv` from a home directory.

## How to check

Find the exact reason in error.log:

```bash
tail -n 30 /var/log/nginx/error.log | grep -E 'forbidden|Permission denied|access forbidden'
```

Show permissions along the whole path:

```bash
namei -l /var/www/site/index.html
```

Test as the nginx user whether the file is readable:

```bash
sudo -u www-data cat /var/www/site/index.html >/dev/null && echo ok
```

Show root, index and access rules in the active config:

```bash
nginx -T 2>/dev/null | grep -nE '^\s*(root|index|autoindex|deny|allow|auth_basic) '
```

## Fix

1. Create an index file or set `index index.php index.html;` in the server block.
2. Give directories 755, files 644 and the right owner; every parent directory needs x for the nginx user.
3. With SELinux restore the context: `restorecon -Rv /var/www/site`.
4. Review deny/allow rules, especially in `location` blocks that match several paths.
5. After changes run `nginx -t && systemctl reload nginx`.

## FAQ

**What is the difference between 401 and 403?**

401 means authentication is missing or invalid – with credentials it may work. 403 means the server refuses access regardless of login.

**Should I enable autoindex to get rid of the 403?**

Only if you really want a file listing. Otherwise add an index file – autoindex exposes every file in the directory.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-403-forbidden-nginx/](https://daniele-michelin.com/en/errors/http-403-forbidden-nginx/)**
