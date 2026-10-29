# Postfix: Host or domain name not found. Name service error for name=… type=MX

> Postfix could not look up the recipient domain in DNS. Either your server's resolver does not work (often a chrooted Postfix without `resolv.conf`), or the domain itself has a problem.

Example:

```
postfix/smtp[2314]: 4F1A2B3C4D: to=<kunde@example.net>, relay=none, delay=0.5, dsn=4.4.3, status=deferred (Host or domain name not found. Name service error for name=example.net type=MX: Host not found, try again)
```

## Common causes

- The DNS resolver is unreachable or misconfigured: `/etc/resolv.conf`, systemd-resolved or a blocked port 53.
- Postfix runs in a chroot and does not find a copy of `resolv.conf` there.
- The recipient domain has a typo, no MX or A record, or its name servers fail.
- A local firewall blocks outgoing DNS queries.
- A short-term DNS outage: “try again” means the error is temporary.

## How to check

Does the lookup work from the shell?:

```bash
dig MX example.net +short
```

Which resolver is configured?:

```bash
cat /etc/resolv.conf
```

Does the Postfix chroot have its own copy?:

```bash
ls -l /var/spool/postfix/etc/resolv.conf
```

Related Postfix settings:

```bash
postconf -n | grep -E 'disable_dns_lookups|smtp_host_lookup|myhostname'
```

## Fix

1. Fix the resolver, or install a local caching one such as unbound, so DNS no longer fails.
2. Refresh the chroot copy and restart: `cp /etc/resolv.conf /var/spool/postfix/etc/resolv.conf && systemctl restart postfix`.
3. Check the recipient address for typos and the domain with `dig MX`.
4. Allow outgoing UDP and TCP port 53 in the firewall.
5. After the fix, retry the queue: `postqueue -f`.

## FAQ

**Is “try again” a permanent error?**

No. It is a temporary failure (4.x.x). Postfix keeps the message and tries again for several days.

**How do I check my own domain's mail DNS?**

Use the free [mail DNS checker](https://daniele-michelin.com/en/tools/mail-dns-checker/) for MX, SPF, DKIM, DMARC and PTR.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postfix-host-or-domain-name-not-found-name-service-error/](https://daniele-michelin.com/en/errors/postfix-host-or-domain-name-not-found-name-service-error/)**
