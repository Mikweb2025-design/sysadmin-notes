# certbot renew: Failed to renew certificate (all renewals failed)

> Let's Encrypt certificates last a short time, so renewal must work on its own. When `certbot renew` fails, the certificate keeps ageing until the browser shows an expiry warning. The cause is usually in the saved renewal settings, not in Let's Encrypt.

Example:

```
Failed to renew certificate cloud.example.com with error: Some challenges have failed.
All renewals failed. The following certificates could not be renewed:
  /etc/letsencrypt/live/cloud.example.com/fullchain.pem (failure)
```

## Common causes

- The renewal file uses `authenticator = standalone`, but nginx or Apache already holds port 80.
- The webroot path or the web server changed, so the challenge file is no longer reachable on the domain.
- The renewal timer or cron job is not running, so renewal is never attempted.
- A `pre`, `post` or `deploy` hook fails, for example the reload of a service that no longer exists.
- DNS, firewall or an IPv6 record no longer reaches this server for the domain.

## How to check

Which certificates exist and when do they expire?:

```bash
sudo certbot certificates
```

Test the full renewal without changing anything:

```bash
sudo certbot renew --dry-run
```

Is a renewal timer scheduled?:

```bash
systemctl list-timers | grep -i certbot
```

What does the log say?:

```bash
sudo tail -n 40 /var/log/letsencrypt/letsencrypt.log
```

## Fix

1. Run `sudo certbot renew --dry-run` and read the first error. It uses the staging server, so it is safe and does not use up the production limits.
2. If it says the port is in use, change the method: `sudo certbot certonly --nginx --cert-name cloud.example.com -d cloud.example.com` (or `--apache` or `--webroot -w /var/www/html`). This rewrites the renewal file.
3. Enable the timer: `sudo systemctl enable --now certbot.timer`. With the snap package it is `snap.certbot.renew.timer`.
4. Fix or remove failing hooks in `/etc/letsencrypt/renewal-hooks/`, and use a `deploy` hook such as `systemctl reload nginx` to load the new certificate.
5. Run the dry run again until it ends with success, then check the new expiry with `sudo certbot certificates`.

## FAQ

**Is --force-renewal the answer?**

Rarely. It does not fix the cause and each forced issue counts against the rate limits. Fix the failing step found by the dry run first.

**The renewal worked but the browser still shows the old certificate. Why?**

The web server loads the certificate only when it starts or reloads. Add a deploy hook that reloads it, or reload by hand once.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/certbot-failed-to-renew-certificate-renewals-failed/](https://daniele-michelin.com/en/errors/certbot-failed-to-renew-certificate-renewals-failed/)**
