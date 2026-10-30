# nginx: connect() to unix:/run/php/php-fpm.sock failed (13: Permission denied) while connecting to upstream

> nginx found the PHP-FPM socket but is not allowed to open it. The visitor gets **502 Bad Gateway**. The owner, group or mode of the socket do not match the user nginx runs as.

Example:

```
2026/10/29 10:03:17 [crit] 1234#1234: *321 connect() to unix:/run/php/php8.3-fpm.sock failed (13: Permission denied) while connecting to upstream, client: 203.0.113.5, server: example.com, request: "GET /index.php HTTP/2.0", upstream: "fastcgi://unix:/run/php/php8.3-fpm.sock:"
```

## Common causes

- The nginx worker user (`user` in `nginx.conf`) differs from `listen.owner` and `listen.group` of the PHP-FPM pool.
- `listen.mode` in the pool is too strict, so the group cannot write to the socket.
- A separate pool for a site runs as its own user and its socket is owned by that user only.
- A parent directory of the socket cannot be entered by the nginx user.
- The socket path in `fastcgi_pass` points to a socket owned by another service.

## How to check

Owner, group and mode of the socket:

```bash
ls -l /run/php/*.sock
```

Which user does nginx run as?:

```bash
grep -E '^\s*user' /etc/nginx/nginx.conf
```

Pool settings for the socket:

```bash
grep -rEn '^(listen|listen\.owner|listen\.group|listen\.mode|user|group) *=' /etc/php/*/fpm/pool.d/
```

Can every directory on the path be entered?:

```bash
namei -l /run/php/php-fpm.sock
```

## Fix

1. In the pool file under `/etc/php//fpm/pool.d/` set `listen.owner = www-data`, `listen.group = www-data` and `listen.mode = 0660`, matching the nginx user.
2. Alternatively add the nginx user to the group that owns the socket.
3. Make sure `fastcgi_pass` uses the exact socket path of this pool.
4. Restart the PHP-FPM service of your version (its name contains the version), then check the socket again with `ls -l`.
5. Test nginx: `nginx -t && systemctl reload nginx`.

## FAQ

**Should I set the socket to 0666?**

No. That lets every local user talk to PHP-FPM. Use the matching owner and group with mode 0660.

**How is this different from Primary script unknown?**

Here nginx cannot reach PHP-FPM at all. With Primary script unknown the connection works but PHP cannot find the script file.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-php-fpm-socket-13-permission-denied/](https://daniele-michelin.com/en/errors/nginx-php-fpm-socket-13-permission-denied/)**
