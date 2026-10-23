# SMTP 451 4.7.1: greylisting and temporary rejection

> The receiver deliberately rejects the first delivery attempt from an unknown sender; a properly working mail server retries after a few minutes and gets through.

Example:

```
451 4.7.1 <info@example.com>: Recipient address rejected: Greylisted for 300 seconds
```

## Common causes

- First contact with a new sender: Postgrey or Rspamd greylisting works as designed.
- Applications or devices (scanners, ticket systems) send directly without a queue and never retry.
- Large senders use IP pools, so each retry comes from a different IP and counts as new again.
- Other 4.7.1 policies, e.g. rate limits or temporary policy server errors.

## How to check

Find greylisting entries in the log:

```bash
journalctl -u postfix --since today | grep -i greylist
```

Check whether greylisting is in the restrictions:

```bash
postconf smtpd_recipient_restrictions
```

Check Postgrey status:

```bash
systemctl status postgrey
```

On the sending side: is the message still queued?:

```bash
postqueue -p | grep -B2 'Greylisted'
```

## Fix

1. Wait: normally the message arrives after 5–15 minutes, no action needed.
2. Whitelist important partners, for Postgrey in `/etc/postgrey/whitelist_clients.local`, then `systemctl reload postgrey`.
3. Make applications send through a local MTA with a queue (Postfix as relay) instead of direct SMTP.
4. Lower the greylisting delay (Postgrey `--delay=60`) or enable it in Rspamd only for poorly scored mail.

## FAQ

**Is 451 4.7.1 a problem on my side?**

Usually not. Greylisting is the receiver's choice. It only becomes a problem if your system does not retry.

**Does greylisting still make sense today?**

It slows down simple spam bots but also delays legitimate mail like login codes. Many admins now use it only for mail with a medium spam score.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-451-4-7-1-greylisting/](https://daniele-michelin.com/en/errors/smtp-451-4-7-1-greylisting/)**
