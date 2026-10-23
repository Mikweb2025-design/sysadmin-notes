# Let's Encrypt: too many certificates already issued

> Let's Encrypt limits how many certificates are issued in a time window. The error names the limit that was hit and when you can try again (“retry after”).

Example:

```
Error creating new order :: too many certificates (5) already issued for this exact set of identifiers in the last 168h0m0s, retry after 2026-10-25 14:02:11 UTC
```

## Common causes

- More than 5 certificates for exactly the same set of names within 7 days.
- More than 50 new certificates per registered domain in 7 days, e.g. with many subdomains.
- A script or container requests a new certificate on every start because `/etc/letsencrypt` is not persistent.
- Testing against the production API instead of staging.
- Related: “too many failed authorizations” after several failed validations.

## How to check

Show locally available certificates:

```bash
certbot certificates
```

How often was it issued recently? Check the archive files:

```bash
ls -l /etc/letsencrypt/archive/example.com/
```

View publicly issued certificates in CT logs:

```bash
curl -s 'https://crt.sh/?q=example.com&output=json' | jq -r '.[] | .not_before + "  " + .name_value' | sort -r | head -n 20
```

Find the exact error and retry-after in the certbot log:

```bash
grep -iE 'rateLimited|too many' /var/log/letsencrypt/letsencrypt.log | tail -n 5
```

## Fix

1. Wait until the stated “retry after” – the limit cannot be reset by hand.
2. Keep using the existing valid certificate instead of requesting a new one.
3. In containers make `/etc/letsencrypt` a persistent volume.
4. Run tests only with `--dry-run` or `--staging`.
5. For many subdomains consider a wildcard certificate via DNS-01.

## FAQ

**Can I get around the limit by adding a name?**

A different set of names is not covered by the exact-set limit, but still counts towards the per-domain limit. Better fix the cause of the loop.

**Do renewals count?**

Renewals of the same set count towards the duplicate-certificate limit. Normal renewal every 60 days never hits it – only loops do.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/letsencrypt-too-many-certificates-already-issued/](https://daniele-michelin.com/en/errors/letsencrypt-too-many-certificates-already-issued/)**
