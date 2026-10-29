# 550 5.7.1 Unauthenticated email is not accepted due to domain's DMARC policy

> The recipient checked your domain's DMARC policy, and the message passed neither SPF nor DKIM in a way that matches the From domain, so it was rejected as the policy says (`p=reject` or `quarantine`).

Example:

```
550 5.7.1 Unauthenticated email from example.com is not accepted due to domain's DMARC policy. Please contact the administrator of example.com domain if this was a legitimate mail.
```

## Common causes

- A service sends mail for your domain (newsletter tool, CRM, ticket system) but is not in your SPF and does not sign with your domain's DKIM key.
- Alignment fails: SPF or DKIM passes, but for the provider's domain, not for the domain in the From address.
- A forwarder or mailing list changes the message or the sending server, which breaks SPF and DKIM.
- The policy was set to `p=reject` before all senders were ready.
- The SPF record is broken, for example over the lookup limit, so SPF fails for everyone.

## How to check

What is your DMARC policy?:

```bash
dig +short TXT _dmarc.example.com
```

What does your SPF record contain?:

```bash
dig +short TXT example.com | grep spf1
```

Which result did the recipient see? Look in the bounce or the headers:

```bash
grep -iE 'authentication-results|dmarc=|spf=|dkim=' bounce.eml
```

Is the DKIM key for the sending selector published?:

```bash
dig +short TXT selector1._domainkey.example.com
```

## Fix

1. Add the sending service to your SPF record, and keep it under 10 DNS lookups. The [SPF checker](https://daniele-michelin.com/en/tools/spf-lookup-checker/) counts them.
2. Enable DKIM for your own domain at the service, so it signs with `d=example.com` and not with its own domain.
3. Make sure the From address uses the same domain that SPF or DKIM authenticates (alignment).
4. If senders are not ready, go back to `p=none` or `quarantine` and read the aggregate reports (`rua`) first.
5. For forwarded mail, rely on DKIM, which survives forwarding, not on SPF.

## FAQ

**Why does it only fail for some recipients?**

Only receivers that enforce DMARC reject. Others, such as small servers, may ignore your policy.

**How do I move to p=reject safely?**

Start with `p=none` and a report address, fix every legitimate sender from the reports, then go to `quarantine` and only later to `reject`. The [DMARC generator](https://daniele-michelin.com/en/tools/dmarc-record-generator/) builds the record.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dmarc-unauthenticated-email-not-accepted-dmarc-policy/](https://daniele-michelin.com/en/errors/dmarc-unauthenticated-email-not-accepted-dmarc-policy/)**
