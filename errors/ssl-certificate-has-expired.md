# SSL: certificate has expired / ERR_CERT_DATE_INVALID

> The served certificate is outside its validity period. Often it was renewed long ago, but the web server still serves the old file from memory.

Example:

```
curl: (60) SSL certificate problem: certificate has expired
```

## Common causes

- Automatic renewal failed, e.g. port 80 blocked or DNS changed.
- The certificate was renewed, but nginx, Postfix or Dovecot was not reloaded.
- The certbot timer or cron job no longer exists, e.g. after a migration.
- The certificate was copied by hand and lives outside `/etc/letsencrypt/live`.
- The client clock is wrong – then even a valid certificate looks expired.

## How to check

Show the validity of the certificate actually served:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -dates
```

Show expiry dates of certbot certificates:

```bash
certbot certificates 2>/dev/null | grep -E 'Certificate Name|Expiry'
```

Check that the renewal timer is active:

```bash
systemctl list-timers | grep -i certbot
```

Test renewal without fetching real certificates:

```bash
certbot renew --dry-run
```

## Fix

1. Renew now: `certbot renew` and reload the service.
2. Make the reload permanent: `--deploy-hook 'systemctl reload nginx'` or a script in `/etc/letsencrypt/renewal-hooks/deploy/`.
3. Fix errors from `certbot renew --dry-run`, typically firewall, DNS or changed vhosts.
4. Monitor expiry dates – a warning 14 days before expiry is enough.
5. On mail servers remember that Postfix and Dovecot need a reload too.

## FAQ

**Why is the certificate expired although certbot renewed it?**

The service keeps the old certificate in memory. Without a reload nginx keeps serving the old one even though the file on disk is new.

**Can an intermediate certificate expire?**

Yes. If the server sends an outdated chain, clients fail despite a valid server certificate. Always use the current `fullchain.pem`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssl-certificate-has-expired/](https://daniele-michelin.com/en/errors/ssl-certificate-has-expired/)**
