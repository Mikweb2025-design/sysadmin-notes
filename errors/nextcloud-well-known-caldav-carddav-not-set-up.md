# Nextcloud: Your web server is not properly set up to resolve "/.well-known/caldav"

> Calendar and contact apps find your server through the `/.well-known/caldav` and `/.well-known/carddav` addresses. The overview warns when these do not redirect to `/remote.php/dav`. Sync still works if you enter the full DAV URL by hand.

Example:

```
Your web server is not properly set up to resolve "/.well-known/caldav". Further information can be found in the documentation.
Your web server is not properly set up to resolve "/.well-known/carddav".
```

## Common causes

- The nginx server block has no redirect for the two `.well-known` paths.
- A reverse proxy in front of Nextcloud answers `/.well-known/` itself or does not pass it on.
- On Apache, `.htaccess` is ignored because `AllowOverride` is not `All` or `mod_rewrite` is off, or the file is outdated.
- Nextcloud is installed in a subfolder, so the well-known path must sit on the domain root.

## How to check

What does caldav answer?:

```bash
curl -sI https://cloud.example.com/.well-known/caldav
```

What does carddav answer?:

```bash
curl -sI https://cloud.example.com/.well-known/carddav
```

Is there a rule in the nginx config?:

```bash
nginx -T 2>/dev/null | grep -n well-known
```

Which URL does Nextcloud assume?:

```bash
sudo -u www-data php occ config:system:get overwrite.cli.url
```

## Fix

1. In nginx, add to the server block: `location = /.well-known/carddav { return 301 /remote.php/dav/; }` and the same for `caldav`. Then `nginx -t` and `systemctl reload nginx`.
2. With a reverse proxy, put the two redirects on the proxy, or pass `/.well-known/` through to Nextcloud.
3. On Apache, enable `mod_rewrite` (`a2enmod rewrite`), set `AllowOverride All` for the Nextcloud folder, then run `sudo -u www-data php occ maintenance:update:htaccess`.
4. Check again with `curl -sI`: you want `301` and a `Location` ending in `/remote.php/dav`. Then reload the overview page.

## FAQ

**Does this block calendar and contact sync?**

Not if you enter the full address `https://cloud.example.com/remote.php/dav` in the client. Without the redirect, automatic discovery by just the domain name fails.

**Is a 302 or a 200 also fine?**

The check expects a redirect to `/remote.php/dav`. A 200 with a page body or a 404 means the path is not handled, so the warning stays.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-well-known-caldav-carddav-not-set-up/](https://daniele-michelin.com/en/errors/nextcloud-well-known-caldav-carddav-not-set-up/)**
