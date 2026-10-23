# Postfix warning: hostname … does not resolve to address

> Postfix found a name for the connecting server's IP via PTR, but that name does not point back to the same IP. The client is then treated as “unknown”.

Example:

```
warning: hostname mail.example.net does not resolve to address 203.0.113.25: Name or service not known
```

## Common causes

- The sender's PTR points to a name without a matching A or AAAA record.
- Typical for spam bots and residential lines – then the warning is expected.
- Your own server: PTR set at the hoster, but the A record is missing or points elsewhere.
- IPv6 connection while only an IPv4 reverse entry exists.
- The local resolver returns stale answers or none at all.

## How to check

Look up the PTR of the connecting IP:

```bash
dig -x 203.0.113.25 +short
```

Check that the name points back to the same IP:

```bash
dig +short A mail.example.net; dig +short AAAA mail.example.net
```

Count the most frequent IPs in the log:

```bash
grep 'does not resolve to address' /var/log/mail.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head
```

Check whether strict hostname checks are enabled:

```bash
postconf smtpd_client_restrictions smtpd_recipient_restrictions | grep -o 'reject_unknown[a-z_]*'
```

## Fix

1. If it concerns foreign IPs, the warning is normal and can be ignored.
2. If it is your own server: set the PTR at your provider and create an A or AAAA record with exactly that IP.
3. Prefer `reject_unknown_reverse_client_hostname` over `reject_unknown_client_hostname` – the strict one also hits legitimate servers.
4. For IPv6 sending set a PTR for the IPv6 address, or send over IPv4 only (`smtp_address_preference = ipv4`).

## FAQ

**Is mail rejected because of this warning?**

No, the warning alone rejects nothing. Mail is only rejected if you set `reject_unknown_client_hostname` – then with 450 4.7.25.

**Is this a problem with Gmail or Microsoft?**

For your own outgoing mail, yes: large providers expect matching forward and reverse DNS and otherwise file the mail as spam or reject it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postfix-hostname-does-not-resolve-to-address/](https://daniele-michelin.com/en/errors/postfix-hostname-does-not-resolve-to-address/)**
