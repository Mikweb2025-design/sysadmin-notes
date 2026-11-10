# Port 25 blocked: SMTP timeout on direct delivery

> Your mail server cannot reach the recipient's MX on port 25 because outbound 25 is blocked. Messages stay queued and are retried.

Example:

```
Nov  9 08:14:22 mail postfix/smtp[2211]: connect to gmail-smtp-in.l.google.com[142.250.27.26]:25: Connection timed out
```

## Common causes

- The provider blocks outbound port 25 on residential and standard connections to limit spam.
- Cloud providers block 25 by default and only lift it on request.
- The firewall allows 587 and 465 but not 25.
- The destination blocks your IP, for example due to a missing PTR or a blacklist entry.
- A missing IPv6 route makes the attempt hang instead of failing fast.

## How to check

Test outbound port 25:

```bash
nc -vz -w 5 gmail-smtp-in.l.google.com 25
```

Inspect the mail queue:

```bash
mailq | tail -n 20
```

Typical errors in the log:

```bash
journalctl -u postfix --since '1 hour ago' --no-pager | grep -iE 'timed out|connection refused|deferred' | tail -n 20
```

Public IP and PTR:

```bash
dig +short myip.opendns.com @resolver1.opendns.com; dig +short -x $(dig +short myip.opendns.com @resolver1.opendns.com)
```

## Fix

1. Instead of delivering directly to MX hosts on 25, use submission on port 587 with STARTTLS and authentication: `relayhost = [smtp.provider.tld]:587`.
2. On clouds, request that port 25 be unblocked if direct delivery is really required.
3. Set PTR, SPF and DKIM correctly, otherwise even allowed delivery is rejected.
4. Allow 587 and 465 outbound in the firewall and block 25 deliberately if you do not need it.
5. Watch the queue and flush it with `postqueue -f` after switching to the relay.

## FAQ

**Can I work around port 25?**

Yes, through the provider's submission port with authentication or through a relay. Direct MX delivery needs 25.

**Why do some messages still arrive?**

Because they go through a relay or another port. Only direct delivery to MX hosts on port 25 fails.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-port-25-blocked/](https://daniele-michelin.com/en/errors/smtp-port-25-blocked/)**
