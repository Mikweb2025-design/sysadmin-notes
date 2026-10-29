# Nextcloud: Too many requests. There were too many requests from your network

> Nextcloud's brute-force protection has slowed down or blocked your IP address after repeated failed logins. Often the real cause is a proxy that makes all users look like one address.

Example:

```
Too many requests
There were too many requests from your network. Retry later or contact your administrator if this is an error.
```

## Common causes

- Several wrong passwords from the same address, for example a sync client or a phone with an old password.
- A reverse proxy or Cloudflare hides the real addresses, so every user shares one IP (see `trusted_proxies`).
- Many users sit behind one office address (NAT).
- A script or app polls with wrong credentials.
- An attack really is running against the login.

## How to check

What does the log say about the address?:

```bash
grep -i bruteforce /var/www/nextcloud/data/nextcloud.log | tail -n 10
```

Does Nextcloud see the real client address? Check the trusted proxies:

```bash
sudo -u www-data php occ config:system:get trusted_proxies
```

What does the web server log show for failed logins?:

```bash
grep 'POST /login' /var/log/nginx/access.log | tail -n 20
```

## Fix

1. Reset the address: `sudo -u www-data php occ security:bruteforce:reset 203.0.113.5`.
2. Fix the proxy setup so Nextcloud sees real addresses, see [reverse proxy header](https://daniele-michelin.com/en/errors/nextcloud-reverse-proxy-header-configuration-incorrect/).
3. Whitelist a trusted network such as the office: `sudo -u www-data php occ config:app:set bruteForce whitelist_0 --value=192.0.2.0/24`.
4. Find the client with the old password (sync client, phone, script) and update it, preferably with an app password.
5. Keep the protection on. Do not switch it off just to silence the message.

## FAQ

**How long does the block last?**

The delay grows with the number of failed attempts and fades after a while without new ones. Resetting the address removes it at once.

**Why does it hit everyone at once?**

If the proxy's address is what Nextcloud sees, all users count as one visitor and share the delay.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-too-many-requests-brute-force-protection/](https://daniele-michelin.com/en/errors/nextcloud-too-many-requests-brute-force-protection/)**
