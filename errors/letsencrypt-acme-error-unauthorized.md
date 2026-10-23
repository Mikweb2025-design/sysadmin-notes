# Let's Encrypt: urn:ietf:params:acme:error:unauthorized

> Let's Encrypt could not fetch the challenge file under `/.well-known/acme-challenge/` from the right server and therefore refuses to issue the certificate.

Example:

```
Type: unauthorized Detail: 2001:db8::10: Invalid response from http://example.com/.well-known/acme-challenge/kX3v9...: 404
```

## Common causes

- An AAAA record points to an IPv6 address where no vhost or the wrong one answers – Let's Encrypt prefers IPv6.
- DNS still points to the old server (migration, high TTL).
- Rewrite or redirect rules (Nextcloud, WordPress, forced HTTPS to another host) catch the path.
- Wrong `--webroot` path, so the file is not where nginx serves it.
- A CDN, proxy or firewall blocks port 80.

## How to check

Check A and AAAA records:

```bash
dig +short A example.com; dig +short AAAA example.com
```

Test the challenge path over IPv6:

```bash
curl -6 -sI http://example.com/.well-known/acme-challenge/test
```

Is nginx listening on IPv6 port 80?:

```bash
nginx -T 2>/dev/null | grep -E 'listen .*\[::\]:80'
```

Test issuance without hitting rate limits:

```bash
certbot certonly --dry-run --webroot -w /var/www/letsencrypt -d example.com
```

## Fix

1. Add `listen [::]:80;` to the HTTP vhost, or delete a wrong AAAA record.
2. Serve the challenge path before any redirect: `location ^~ /.well-known/acme-challenge/ { root /var/www/letsencrypt; }`.
3. Lower the TTL before a migration and request the certificate only after the switch.
4. If port 80 is not reachable, switch to the DNS-01 challenge (e.g. a certbot DNS plugin).

## FAQ

**Why does it work in my browser but not for Let's Encrypt?**

Your browser may use IPv4 or a cache. Let's Encrypt queries over IPv6 as soon as an AAAA record exists, and validates from several locations.

**How many failed attempts are allowed?**

Let's Encrypt allows 5 failed validations per account, hostname and hour. Test with `--dry-run` against staging first.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/letsencrypt-acme-error-unauthorized/](https://daniele-michelin.com/en/errors/letsencrypt-acme-error-unauthorized/)**
