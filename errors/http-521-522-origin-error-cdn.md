# Error 521 (Web server is down) and 522 (Connection timed out) behind a CDN

> These codes come from the CDN (for example Cloudflare), not from your server. The CDN reached its own edge but could not talk to your origin: **521** means the connection was refused, **522** means it timed out.

Example:

```
Error 522
Connection timed out
The initial connection between Cloudflare's network and the origin web server timed out.

Error 521
Web server is down
The origin web server refused the connection.
```

## Common causes

- A firewall, fail2ban or rate limit on the origin blocks the CDN's IP addresses (very common with 522).
- The web server on the origin is stopped, crashed or listens on another port (521).
- The DNS record points to an old or wrong origin IP, for example after a migration.
- The origin is overloaded and does not accept new connections in time.
- The CDN connects on a port or protocol the origin does not offer (HTTPS to a server that only listens on 80).

## How to check

Is the web server running and listening?:

```bash
systemctl is-active nginx; ss -tlnp | grep -E ':(80|443)\b'
```

Does the origin answer directly? (use the real origin IP):

```bash
curl -sI --resolve example.com:443:203.0.113.10 https://example.com/ | head -n 5
```

Do requests from the CDN reach the origin at all?:

```bash
sudo tail -n 20 /var/log/nginx/access.log
```

Is the CDN's address range blocked by a ban?:

```bash
sudo fail2ban-client status
```

## Fix

1. Start or repair the origin web server and make sure it listens on the ports the CDN uses.
2. Allow the CDN's published IP ranges in the origin firewall and exclude them from fail2ban and rate limits.
3. Restore the real visitor IP so bans hit visitors, not the CDN: use `set_real_ip_from` for the CDN ranges and `real_ip_header CF-Connecting-IP;` (Cloudflare).
4. Fix the origin IP in the CDN's DNS record and align the CDN SSL mode with what the origin serves.
5. If the origin is overloaded, look at PHP-FPM and database load before blaming the CDN.

## FAQ

**What is the difference between 521 and 522?**

521: the origin actively refused the connection, so nothing listens or a firewall rejects it. 522: no answer in time, which fits a firewall that drops packets or an overloaded server.

**How is this different from 502 and 504?**

502 and 504 come from your own server or proxy reaching its backend. 521 and 522 mean the CDN could not even reach your server.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/http-521-522-origin-error-cdn/](https://daniele-michelin.com/en/errors/http-521-522-origin-error-cdn/)**
