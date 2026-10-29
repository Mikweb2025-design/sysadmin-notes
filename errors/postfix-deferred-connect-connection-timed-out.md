# Postfix: status=deferred (connect to …:25: Connection timed out)

> Your mail server cannot open a connection to the recipient's server on port 25, so it keeps the message in the queue and tries again later. Very often the cause is a blocked outgoing port 25.

Example:

```
postfix/smtp[2314]: 4F1A2B3C4D: to=<kunde@example.net>, relay=mail.example.net[203.0.113.25]:25, delay=312, delays=0.1/0/312/0, dsn=4.4.1, status=deferred (connect to mail.example.net[203.0.113.25]:25: Connection timed out)
```

## Common causes

- The hosting or cloud provider blocks outgoing port 25 by default, which is common on new cloud VMs and home connections.
- A firewall on your server or in front of it drops outgoing SMTP.
- The recipient's server is down or too slow.
- IPv6 is tried first, but your server has no working IPv6 route.
- A wrong or outdated MX record sends you to an address that does not answer.

## How to check

What is in the queue and why?:

```bash
postqueue -p | head -n 20
```

Can your server reach the recipient's port 25? (5 seconds):

```bash
nc -vz -w 5 mail.example.net 25
```

Relay and IP protocol settings:

```bash
postconf -n | grep -E 'relayhost|inet_protocols'
```

Recent timeouts in the log:

```bash
journalctl -u postfix --since '1 hour ago' --no-pager | grep -i 'timed out' | tail -n 10
```

## Fix

1. If the provider blocks port 25, ask to have it unblocked, or send through a smarthost: `relayhost = [smtp.provider.example]:587` with SASL login.
2. If IPv6 does not work, restrict Postfix: `inet_protocols = ipv4`, then `postfix reload`.
3. Allow outgoing TCP 25 (or 587 for the relay) in your firewall.
4. Check the recipient's MX with `dig MX example.net +short` and try the other MX hosts.
5. After the fix, retry the queue: `postqueue -f`.

## FAQ

**Will my messages be lost?**

No. Deferred means Postfix retries. By default it keeps trying for five days and then sends a bounce to the sender.

**How can I check my mail setup in general?**

Use the free [mail DNS checker](https://daniele-michelin.com/en/tools/mail-dns-checker/) for MX, SPF, DKIM, DMARC and PTR.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postfix-deferred-connect-connection-timed-out/](https://daniele-michelin.com/en/errors/postfix-deferred-connect-connection-timed-out/)**
