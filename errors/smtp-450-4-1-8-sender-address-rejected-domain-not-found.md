# SMTP: 450 4.1.8 Sender address rejected: Domain not found

> The receiving server looked up the domain in the sender address and found neither an A nor an MX record, so it refuses the mail with `reject_unknown_sender_domain`. Typical sources are servers that send as `root@hostname.localdomain`.

Example:

```
postfix/smtpd[3105]: NOQUEUE: reject: RCPT from unknown[203.0.113.30]: 450 4.1.8 <root@server1.localdomain>: Sender address rejected: Domain not found; from=<root@server1.localdomain> to=<admin@example.com> proto=ESMTP
```

## Common causes

- The sender address uses a made-up domain such as `.local` or `.localdomain`, often from cron or monitoring mail.
- The sender domain really has no A or MX record, because of a typo or a domain not yet set up.
- The DNS of the receiving server is broken or slow, so a good domain looks missing. The 4xx code means a temporary refusal.
- A new or moved domain whose DNS has not propagated or was deleted by mistake.

## How to check

Does the sender domain have an MX record?:

```bash
dig +short MX example.com
```

Does it have an A record?:

```bash
dig +short A example.com
```

What is the DNS status for the domain?:

```bash
dig example.com MX | grep -E 'status:'
```

Which sender checks are active?:

```bash
postconf -n | grep -E 'smtpd_sender_restrictions|unknown_address_reject_code|myorigin'
```

## Fix

1. If the domain is fake, change the sender. For your own server, set a real domain: `myorigin = example.com` in `main.cf`, or rewrite the sender with `sender_canonical_maps`.
2. For apps and cron, set the From address in their own settings to a real address in a domain that has DNS records.
3. Fix missing DNS: publish an A or MX record for the sender domain, and check the zone for typos.
4. If your own resolver is the problem, test it with `dig example.com MX @1.1.1.1` and repair `/etc/resolv.conf` or the local resolver.
5. Only as a last resort, remove `reject_unknown_sender_domain` from `smtpd_sender_restrictions`. That check blocks a lot of junk.

## FAQ

**Why is it a 450 and not a 550?**

Postfix uses `unknown_address_reject_code`, which defaults to 450 (try again). Senders retry, so a short DNS problem does not lose mail. Set it to 550 only if you want a hard refusal.

**Will the mail still arrive?**

A sending server keeps retrying for days. If the sender domain is fake, it will never succeed, so fix the address at the source.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-450-4-1-8-sender-address-rejected-domain-not-found/](https://daniele-michelin.com/en/errors/smtp-450-4-1-8-sender-address-rejected-domain-not-found/)**
