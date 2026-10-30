# Postfix: mail for example.com loops back to myself

> Postfix looked up where to send the mail and ended up at itself, so it stops instead of looping forever. The routing says this host is responsible, but the local configuration does not accept the domain.

Example:

```
postfix/smtp[1822]: 7B2C14A0F3: to=<info@example.com>, relay=none, delay=0.1, status=bounced (mail for example.com loops back to myself)
```

## Common causes

- The MX record of the domain points to this server, but the domain is not in `mydestination` or `virtual_mailbox_domains`.
- A `relayhost` or a `transport` entry sends the domain to a host name that resolves to this same machine.
- The MX host name or the `relayhost` resolves to an address of this server, for example after a move or a changed DNS record.
- The domain sits in both `mydestination` and a virtual domain list, or `myhostname` differs from the name in the MX.

## How to check

What are the MX records of the domain?:

```bash
dig +short MX example.com
```

What do the MX hosts resolve to?:

```bash
dig +short A mail.example.com
```

Which addresses does this server have?:

```bash
ip -brief address
```

Routing and domain settings:

```bash
postconf -n | grep -E 'myhostname|mydestination|relayhost|virtual_mailbox_domains|transport_maps'
```

## Fix

1. If this server should receive mail for the domain, add the domain to `virtual_mailbox_domains` (or to `mydestination`), but to one list only, and define where the mailboxes are.
2. If another server should receive it, correct the MX record to point there, and remove the entry in `transport_maps` or the `relayhost` that points back to this machine.
3. Make `myhostname` match the real host name, and check that no name in the routing resolves to your own IP unless intended.
4. Apply with `postfix reload`, then retry the stuck messages with `postqueue -f`.
5. Send a test mail and watch `journalctl -u postfix` to see the delivery.

## FAQ

**What is the difference to a hop count exceeded bounce?**

Hop count exceeded (554 5.4.14) appears after a mail has travelled around a loop between servers. Loops back to myself is caught earlier, on the first lookup.

**Does a backup MX cause this?**

It can. A secondary MX that you list for a domain must be set up as a relay for it (`relay_domains` with a transport), or it will refuse or loop the mail.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postfix-mail-for-domain-loops-back-to-myself/](https://daniele-michelin.com/en/errors/postfix-mail-for-domain-loops-back-to-myself/)**
