# SMTP: 554 5.7.1 Relay access denied

> The server is asked to deliver mail to a domain it does not own, from a client it does not trust. Refusing that is correct, it is what stops an open relay. The task is to find out whether your client should have logged in, or your own domain is not set up.

Example:

```
postfix/smtpd[2210]: NOQUEUE: reject: RCPT from client.example.net[203.0.113.20]: 554 5.7.1 <user@example.org>: Relay access denied; from=<me@example.com> to=<user@example.org> proto=ESMTP helo=<client>
```

## Common causes

- The mail client or app does not authenticate, or uses port 25 where login is not offered. Use the submission port 587 with login.
- `smtpd_relay_restrictions` lacks `permit_sasl_authenticated`, so even logged-in users are refused.
- The recipient domain is yours, but it is not listed in `virtual_mailbox_domains`, `mydestination` or `relay_domains`.
- The sending host is a trusted internal machine, but its address is not in `mynetworks`.
- A typo in the recipient domain, or an MX record that points to this server for a domain it should not serve.

## How to check

Relay and recipient rules in effect:

```bash
postconf -n | grep -E 'smtpd_relay_restrictions|smtpd_recipient_restrictions|mynetworks'
```

Which domains does this server accept?:

```bash
postconf -n | grep -E 'mydestination|virtual_mailbox_domains|virtual_alias_domains|relay_domains'
```

Is login enabled on the submission service?:

```bash
postconf -P submission/inet/smtpd_sasl_auth_enable
```

The rejections in the log:

```bash
journalctl -u postfix --since '1 hour ago' --no-pager | grep 'Relay access denied' | tail -n 10
```

## Fix

1. For your users, make them send through port 587 with STARTTLS and a login, not over port 25 without login.
2. Allow logged-in users: `smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, defer_unauth_destination`, and enable `smtpd_sasl_auth_enable = yes` on the submission service in `master.cf`.
3. If the recipient domain should be delivered here, add it to `virtual_mailbox_domains` (or `mydestination`), but to only one of them, then `postfix reload`.
4. For one trusted internal host, add only that address to `mynetworks`. Never add wide ranges or `0.0.0.0/0`.
5. Send a test again and check that the log line is gone.

## FAQ

**Does this mean my server is an open relay?**

No, the opposite. This message is the protection working. You only need to act if your own users or your own domain are being refused.

**Why does it only happen for some recipients?**

Mail to your own domains is accepted, mail to other domains needs authentication. So local recipients work, and external ones get this error until the client logs in.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-554-5-7-1-relay-access-denied/](https://daniele-michelin.com/en/errors/smtp-554-5-7-1-relay-access-denied/)**
