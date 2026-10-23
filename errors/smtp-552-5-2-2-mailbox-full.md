# SMTP 552 5.2.2: mailbox full / quota exceeded

> The receiving server refuses the message because the recipient's mailbox has hit its quota. The sender gets a bounce and the message is not delivered.

Example:

```
552 5.2.2 <anna@example.com>: Quota exceeded (mailbox for user is full)
```

## Common causes

- The mailbox reached the quota limit set in Dovecot.
- Trash and Sent are never emptied.
- Large attachments, or a sync client uploading mail twice.
- Quota rule set too small, e.g. a default value from a template.
- Less common: the filesystem itself is full or out of inodes.

## How to check

Show usage and limit of the mailbox:

```bash
doveadm quota get -u anna@example.com
```

Show size per folder:

```bash
doveadm mailbox status -u anna@example.com 'messages vsize' '*'
```

Show the quota configuration in Dovecot:

```bash
doveconf -n | grep -A5 -E '^plugin|quota'
```

Check space and inodes on the mail storage:

```bash
df -h /var/vmail && df -i /var/vmail
```

## Fix

1. Raise the limit in the quota rule (e.g. `quota_rule = *:storage=10G`) or set it per user via userdb.
2. Empty Trash automatically: `doveadm expunge -u anna@example.com mailbox Trash savedbefore 30d`, as a cron job for all users with `-A`.
3. Set a tolerance with `quota_grace = 10%` so one large message does not bounce right away.
4. Warn users before they hit the limit with `quota_warning`.
5. After changes run `doveadm quota recalc -u anna@example.com`.

## FAQ

**What is the difference between 552 5.2.2 and 452 4.2.2?**

452 is temporary: the sender retries later. 552 is permanent and the message bounces. Dovecot uses the permanent code by default.

**Can the sender do anything about it?**

No. The problem is the recipient's mailbox. The sender can only let them know or resend later.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-552-5-2-2-mailbox-full/](https://daniele-michelin.com/en/errors/smtp-552-5-2-2-mailbox-full/)**
