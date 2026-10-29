# Certbot: Timeout during connect (likely firewall problem)

> Let's Encrypt could not open a connection to your server on port 80 to fetch the challenge file. Something blocks it on the way: a firewall, a wrong address, or an IPv6 address that does not answer.

Example:

```
Certbot failed to authenticate some domains (authenticator: nginx).
Domain: example.com
Type:   connection
Detail: 203.0.113.10: Fetching http://example.com/.well-known/acme-challenge/hkQ3...: Timeout during connect (likely firewall problem)
```

## Common causes

- A firewall or cloud security group blocks incoming port 80.
- The DNS record points to a wrong or old IP address.
- The domain has an AAAA record, but the server is not reachable over IPv6. Let's Encrypt prefers IPv6.
- The server sits behind NAT and port 80 is not forwarded.
- A CDN or proxy in front does not pass the challenge path.

## How to check

Which addresses does the domain have?:

```bash
dig +short A example.com; dig +short AAAA example.com
```

Is port 80 reachable from outside? (run from another machine):

```bash
curl -I http://example.com/.well-known/acme-challenge/test
```

Is anything listening on port 80 here?:

```bash
ss -ltnp | grep ':80 '
```

What does the firewall allow?:

```bash
ufw status verbose
```

## Fix

1. Open port 80, also in the cloud provider's security group: `ufw allow 80/tcp`.
2. Fix the A record, or remove an AAAA record that does not work, or make the server answer on IPv6.
3. Forward port 80 on the router if the server is behind NAT.
4. If port 80 cannot be opened, use the DNS challenge instead: `certbot certonly --manual --preferred-challenges dns -d example.com`.
5. Then renew: `certbot renew --dry-run` to check without using up the rate limit.

## FAQ

**Does port 443 matter for this error?**

No. The HTTP challenge always uses port 80, even if the site runs on 443 only. Keep port 80 open and redirect it to https.

**Why does it only fail for some domains?**

Often one of them has a broken AAAA record. Check each domain with `dig AAAA`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/certbot-timeout-during-connect-firewall/](https://daniele-michelin.com/en/errors/certbot-timeout-during-connect-firewall/)**
