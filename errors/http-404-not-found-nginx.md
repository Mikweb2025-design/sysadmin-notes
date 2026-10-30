# nginx: 404 Not Found

> nginx looked for a file at the path it built from `root` or `alias` and the request URI, and did not find it. The error log shows the exact path it tried. Usually the path, the routing rule or the server block is wrong.

Example:

```
2026/10/29 09:41:03 [error] 1234#1234: *7788 open() "/var/www/example.com/html/about" failed (2: No such file or directory), client: 203.0.113.5, server: example.com, request: "GET /about HTTP/2.0", host: "example.com"
```

## Common causes

- The `root` or `alias` points to the wrong directory, or the deployment did not put the files there.
- An application with pretty URLs (front controller, single-page app) lacks `try_files ... /index.php?$args` or `/index.html`.
- The request reached the wrong `server` block (wrong `server_name` or the `default_server`).
- File names are case-sensitive on Linux: `Logo.PNG` is not `logo.png`.
- A `location` with `alias` has a trailing-slash mismatch, or a regex location overrides the intended one.

## How to check

The exact path nginx tried:

```bash
sudo tail -n 20 /var/log/nginx/error.log
```

The effective rules of the server:

```bash
nginx -T 2>/dev/null | grep -nE 'server_name|root |alias |try_files|location '
```

Does the file exist at that path?:

```bash
ls -l /var/www/example.com/html/about
```

Which server block answers this host? (check the response):

```bash
curl -sI -H 'Host: example.com' http://127.0.0.1/about | head -n 5
```

## Fix

1. Compare the path from the error log with the real file location and correct `root` or `alias`, or deploy the missing files.
2. For front-controller apps use `location / { try_files $uri $uri/ /index.php?$args; }`, for single-page apps `try_files $uri /index.html;`.
3. Check `server_name` so that the host matches the intended block, and set a deliberate `default_server`.
4. Fix the case of file names or links, and with `alias` make the location and alias slashes match.
5. Test and apply: `nginx -t && systemctl reload nginx`.

## FAQ

**How is a 404 different from 403?**

404 means nginx found no matching file. 403 means it found the path but may not read it or may not list the directory, which is a permission or index problem.

**Can a 404 come from the application and not from nginx?**

Yes. If PHP-FPM or a proxy backend answers 404, nginx only passes it on. The error log has no `open() failed` line in that case, so check the application routing.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-404-not-found-nginx/](https://daniele-michelin.com/en/errors/http-404-not-found-nginx/)**
