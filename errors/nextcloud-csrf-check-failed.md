# Nextcloud: CSRF check failed on login or save

> The CSRF token in the form does not match the session on the server. Almost always the session was lost between loading and submitting – not an attack.

Example:

```
CSRF check failed
```

## Common causes

- Session cookie not stored, e.g. a Secure cookie over HTTP behind a TLS proxy.
- Reverse proxy without matching `overwriteprotocol` and `trusted_proxies`.
- PHP cannot write sessions (permissions on `session.save_path`, full disk).
- Redis as session store unreachable or full.
- Login page left open too long so the session expired, or access via two different hostnames.

## How to check

Check whether and how the session cookie is set:

```bash
curl -sI https://cloud.example.com/login | grep -i '^set-cookie'
```

Show the Nextcloud proxy settings:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get overwriteprotocol; sudo -u www-data php /var/www/nextcloud/occ config:system:get trusted_proxies
```

Show PHP session storage:

```bash
php -i | grep -E 'session.save_(handler|path)'
```

Show cache and Redis configuration:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get memcache.distributed; sudo -u www-data php /var/www/nextcloud/occ config:system:get redis
```

## Fix

1. Behind a TLS proxy set `'overwriteprotocol' => 'https'` and add the proxy IP to `trusted_proxies`.
2. Always use the same hostname and set `overwrite.cli.url` to match.
3. Make the session directory writable or check Redis: `redis-cli ping`.
4. Clear browser cookies for the domain and reload the login page.
5. Check disk space – without free space session files fail silently.

## FAQ

**Is “CSRF check failed” an attack?**

In practice almost never. It means the token from the form does not match the current session – usually a lost or new session.

**Why does it only hit some users?**

Often browser extensions, blocked cookies or a bookmark to a different hostname are to blame. A private window narrows it down quickly.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-csrf-check-failed/](https://daniele-michelin.com/en/errors/nextcloud-csrf-check-failed/)**
