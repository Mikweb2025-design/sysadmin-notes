# SMTP 553 5.7.1: sender address rejected: not owned by user

> Postfix compares the sender address (MAIL FROM) with the authenticated SASL user. If `smtpd_sender_login_maps` says the address does not belong to that user, the message is rejected.

Example:

```
553 5.7.1 <info@example.com>: Sender address rejected: not owned by user anna@example.com
```

## Common causes

- `reject_sender_login_mismatch` is active and the sender address is missing from `smtpd_sender_login_maps`.
- Sending from a shared address (info@, noreply@) with a personal login.
- The application sets a different envelope sender than the login address.
- The map only lists the address itself, not aliases or capitalised variants.

## How to check

Show sender restrictions and login map:

```bash
postconf smtpd_sender_restrictions smtpd_sender_login_maps
```

Check which login owns the sender address:

```bash
postmap -q info@example.com $(postconf -h smtpd_sender_login_maps)
```

Show the submission port settings:

```bash
postconf -M submission/inet
```

Find rejected senders in the log:

```bash
journalctl -u postfix --since '1 hour ago' | grep 'Sender address rejected'
```

## Fix

1. For shared addresses add an entry to the login map, e.g. `info@example.com anna@example.com,ben@example.com`.
2. For hash maps run `postmap`, then `postfix reload`.
3. In applications set MAIL FROM to the login address and use the desired address only as Reply-To.
4. Do not just disable the check: without it any logged-in user can send as anyone.

## FAQ

**How is this different from 550 5.7.1?**

553 5.7.1 is about the sender address when you send. 550 5.7.1 is usually a recipient-side policy or relay access denied.

**Can one address belong to several users?**

Yes. The login map accepts several logins separated by commas. With SQL maps the query returns several rows.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-553-5-7-1-sender-address-rejected/](https://daniele-michelin.com/en/errors/smtp-553-5-7-1-sender-address-rejected/)**
