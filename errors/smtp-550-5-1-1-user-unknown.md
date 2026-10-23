# SMTP 550 5.1.1: user unknown / recipient address rejected

> The target server does not know the recipient address and rejects the message permanently. Either the address is wrong or it is missing from the server's tables.

Example:

```
550 5.1.1 <anna@example.com>: Recipient address rejected: User unknown in virtual mailbox table
```

## Common causes

- Typo in the address, or the mailbox was deleted.
- Address missing from `virtual_mailbox_maps`, or alias missing from `virtual_alias_maps`.
- Hash file not rebuilt with `postmap` after editing.
- Domain listed in `mydestination` and in `virtual_mailbox_domains` – Postfix then looks for local Unix users (“User unknown in local recipient table”).
- After a migration the MX still points to the old server, which no longer has the mailboxes.

## How to check

Show Postfix domain classes and maps:

```bash
postconf mydestination virtual_mailbox_domains virtual_mailbox_maps virtual_alias_maps
```

Check whether the address is a known mailbox:

```bash
postmap -q anna@example.com $(postconf -h virtual_mailbox_maps)
```

Check whether an alias exists for the address:

```bash
postmap -q anna@example.com $(postconf -h virtual_alias_maps)
```

Find rejected recipients in the log:

```bash
journalctl -u postfix --since '1 hour ago' | grep 'User unknown'
```

## Fix

1. Have the sender check the address – often it is just a typo.
2. Create the mailbox or alias; for hash maps run `postmap /etc/postfix/virtual` and `postfix reload`.
3. Put every domain in exactly one class: local domains in `mydestination`, virtual ones only in `virtual_mailbox_domains`.
4. After migrations check the MX record and TTL so mail does not land on the old server.
5. Do not add a catch-all as a workaround: it attracts spam and hides real errors.

## FAQ

**Is 550 5.1.1 a spam problem?**

No. The code is only about the recipient. Your reputation is not affected unless you send in bulk to invalid addresses.

**Why does Postfix reject although the user is in the database?**

Usually the SQL query in the map returns nothing, e.g. a typo in the query or the wrong column. Test with `postmap -q` directly against the map file.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-550-5-1-1-user-unknown/](https://daniele-michelin.com/en/errors/smtp-550-5-1-1-user-unknown/)**
