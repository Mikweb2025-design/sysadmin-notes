# SMTP: 550 5.4.1 Recipient address rejected: Access denied

> This bounce comes from Exchange Online when it protects itself against address harvesting. The mail is refused at the edge because it cannot find the recipient in the directory. It looks like a permission problem, but it nearly always means the address does not exist there.

Example:

```
550 5.4.1 Recipient address rejected: Access denied. AS(201806281) [DB3EUR04FT012.eop-eur04.prod.protection.outlook.com]
```

## Common causes

- The recipient address has a typo, or the person or alias was removed or never created.
- The domain is set to "Internal Relay" in Exchange Online, but the mailbox lives on another server, and no connector sends the mail on.
- The mailbox was just created or renamed, and the change has not spread yet.
- The MX of the domain points to Microsoft 365, but the mailboxes are hosted somewhere else, for example after a migration.
- For a distribution group, it accepts only authenticated senders, or the group was deleted.

## How to check

Where does the MX of the recipient domain point?:

```bash
dig +short MX example.com
```

Find the bounce in your own Postfix log:

```bash
journalctl -u postfix --since today --no-pager | grep ' 5.4.1 ' | tail -n 10
```

Does the recipient exist in the tenant? (Exchange Online PowerShell, as admin):

```bash
Get-Recipient user@example.com | Format-List PrimarySmtpAddress,RecipientType
```

What type is the accepted domain? (Exchange Online PowerShell, as admin):

```bash
Get-AcceptedDomain | Format-Table DomainName,DomainType
```

## Fix

1. Check the address letter by letter with the recipient, and test with the primary address.
2. If the mailbox or alias is missing, create it or add the address to the right user. Wait a short time for it to spread.
3. If mailboxes are still on-premises, set the accepted domain to Internal Relay and create the outbound connector, or keep the domain Authoritative when all mailboxes are in Exchange Online.
4. After a migration, move the MX record to the system that now holds the mailboxes.
5. Send the test again from outside and compare the new bounce, if any.

## FAQ

**How is this different from 550 5.1.1?**

5.1.1 says the user is unknown. 5.4.1 with Access denied is the answer of Exchange Online's directory-based edge blocking, which also hides whether an address exists. The cause is still, most of the time, that the recipient is not found.

**Can the sender fix this alone?**

The sender can check the spelling and ask the recipient to confirm the address. If the address is right, only the recipient's admin can fix it in the tenant.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-550-5-4-1-recipient-address-rejected-access-denied/](https://daniele-michelin.com/en/errors/smtp-550-5-4-1-recipient-address-rejected-access-denied/)**
