# Nextcloud: the reverse proxy header configuration is incorrect

> Nextcloud sees forwarded headers (X-Forwarded-For) from a proxy it does not trust, or no proxy is configured although one is in front. Visitors then appear with the wrong IP, or redirects and links use http instead of https.

Example:

```
The reverse proxy header configuration is incorrect. This is a security issue and can allow an attacker to spoof their IP address as visible to the Nextcloud.
```

## Common causes

- Nextcloud runs behind nginx, Apache, Traefik, HAProxy or a Docker proxy, but `trusted_proxies` is not set in `config.php`.
- The proxy does not send `X-Forwarded-For`, `X-Forwarded-Proto` and `Host`.
- `overwriteprotocol` or `overwrite.cli.url` do not match the public address (http against https).
- More than one hop, for example Cloudflare, then nginx, then the Nextcloud container, and only one of them is trusted.
- The opposite case: there is no proxy, but `trusted_proxies` still lists an old address.

## How to check

Which proxies does Nextcloud trust?:

```bash
sudo -u www-data php occ config:system:get trusted_proxies
```

Is the protocol forced correctly?:

```bash
sudo -u www-data php occ config:system:get overwriteprotocol
```

Which headers does the proxy set?:

```bash
grep -rnE 'proxy_set_header|X-Forwarded' /etc/nginx/
```

What address does the app see for you? (Settings → Administration → Overview shows the warning):

```bash
curl -sI https://cloud.example.com/status.php
```

## Fix

1. Add the proxy's address: `sudo -u www-data php occ config:system:set trusted_proxies 0 --value=192.0.2.10`. Use one index per proxy.
2. Force the public protocol: `sudo -u www-data php occ config:system:set overwriteprotocol --value=https`.
3. Make the proxy send the headers: `proxy_set_header Host $host;`, `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;` and `proxy_set_header X-Forwarded-Proto $scheme;`.
4. Behind Cloudflare, add Cloudflare's address ranges to `trusted_proxies` and set `forwarded_for_headers` to `HTTP_CF_CONNECTING_IP`.
5. If there is no proxy, remove `trusted_proxies` and `forwarded_for_headers`.

## FAQ

**Why is this called a security issue?**

Nextcloud only believes the forwarded IP from trusted proxies. Otherwise anyone could send a fake header and hide behind a false address, which also defeats brute-force protection.

**Why does every user show the same IP?**

Because the proxy's address is what Nextcloud sees. Then the brute-force protection can throttle all users at once, see [too many requests](https://daniele-michelin.com/en/errors/nextcloud-too-many-requests-brute-force-protection/).

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-reverse-proxy-header-configuration-incorrect/](https://daniele-michelin.com/en/errors/nextcloud-reverse-proxy-header-configuration-incorrect/)**
