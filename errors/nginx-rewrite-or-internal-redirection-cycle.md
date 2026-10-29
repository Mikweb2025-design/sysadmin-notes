# nginx: rewrite or internal redirection cycle while internally redirecting to "/index.html"

> nginx redirected the request internally again and again and gave up after 10 steps, answering 500. Typically `try_files` falls back to a file that does not exist under the current root.

Example:

```
2026/10/29 09:14:02 [error] 1234#1234: *5678 rewrite or internal redirection cycle while internally redirecting to "/index.html", client: 203.0.113.5, server: example.com, request: "GET /app/ HTTP/2.0"
```

## Common causes

- `try_files $uri $uri/ /index.html;` but `/index.html` does not exist in the `root` (build missing, wrong path).
- The `root` or an `alias` was changed and no longer points to the deployed site.
- The `index` file named in the config is not there.
- A `location` or `rewrite` rule sends the request back into itself.
- A single-page app was deployed without its `index.html`.

## How to check

The effective rules for this site:

```bash
nginx -T 2>/dev/null | grep -nE 'server_name|root |alias |try_files|index '
```

Does the fallback file exist?:

```bash
ls -l /var/www/app/index.html
```

The exact log lines:

```bash
tail -n 20 /var/log/nginx/error.log
```

Is the configuration valid?:

```bash
nginx -t
```

## Fix

1. Point `root` to the directory that really contains `index.html`.
2. Make sure the build was deployed and `index.html` is there and readable by nginx.
3. For a single-page app use `try_files $uri /index.html;` and nothing that falls back to a path that again falls back.
4. With an `alias`, check the trailing slashes and that `try_files` uses a path inside it.
5. Reload: `nginx -t && systemctl reload nginx`.

## FAQ

**Why 500 and not 404?**

nginx stops after 10 internal redirects and reports a server error, because the loop is a configuration problem, not a missing page.

**Can the same message come from a rewrite rule?**

Yes. A `rewrite` without `last`, `break` or a clear exit condition can also loop. Check each `rewrite` in the matching location.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-rewrite-or-internal-redirection-cycle/](https://daniele-michelin.com/en/errors/nginx-rewrite-or-internal-redirection-cycle/)**
