# Nextcloud: access through untrusted domain

> Nextcloud was opened via a hostname or IP that is not listed in `trusted_domains`, so it refuses to log you in.

Example:

```
Access through untrusted domain. Please contact your administrator. If you are an administrator, edit the "trusted_domains" setting in config/config.php.
```

## Common causes

- A new hostname after a domain change or migration that was never added.
- Access directly via the IP address or an internal name.
- The reverse proxy forwards a different `Host` header than the public name.
- Docker: `NEXTCLOUD_TRUSTED_DOMAINS` is only applied on the first install.

## How to check

Show configured trusted domains:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get trusted_domains
```

Check proxy-related settings:

```bash
grep -nE 'trusted_proxies|overwritehost|overwriteprotocol|overwrite.cli.url' /var/www/nextcloud/config/config.php
```

Test the backend directly with a Host header:

```bash
curl -s -o /dev/null -w '%{http_code}\n' -H 'Host: cloud.example.com' http://127.0.0.1/status.php
```

Check which Host header nginx forwards:

```bash
nginx -T 2>/dev/null | grep -n 'proxy_set_header Host'
```

## Fix

1. Add the domain (use the next free index): `sudo -u www-data php occ config:system:set trusted_domains 1 --value=cloud.example.com`.
2. Forward the original host in the reverse proxy: `proxy_set_header Host $host;`.
3. Behind a proxy set `trusted_proxies` and, if needed, `overwritehost` and `overwriteprotocol`.
4. With Docker, change the values in `config.php` or via occ inside the container, not just the environment variable.

## FAQ

**Can I allow every domain with *?**

Wildcards like `*.example.com` work, a bare * makes no sense. The list protects against host header attacks such as manipulated password reset links.

**Do I need to restart anything after the change?**

No. Nextcloud reads `config.php` on every request, so the change applies immediately.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-trusted-domain-error/](https://daniele-michelin.com/en/errors/nextcloud-trusted-domain-error/)**
