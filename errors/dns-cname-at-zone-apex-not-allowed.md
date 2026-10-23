# DNS: CNAME at the apex – “CNAME and other data”

> A CNAME may not coexist with other records of the same name. The zone apex (example.com) always has SOA and NS – so a CNAME is not allowed there.

Example:

```
dns_master_load: example.com.zone:12: example.com: CNAME and other data
```

## Common causes

- A hoster or SaaS provider asks for “CNAME to app.provider.net”, including the root domain.
- The zone editor rejects the entry or BIND refuses to load the zone.
- A provider accepts it anyway – then MX or TXT lookups break unpredictably.
- CNAME and TXT on the same subdomain name, e.g. for a verification.

## How to check

Check the zone file for errors (BIND):

```bash
named-checkzone example.com /etc/bind/db.example.com
```

Show existing records at the apex:

```bash
dig example.com CNAME +short; dig example.com SOA +short; dig example.com MX +short
```

Get the provider's target IP:

```bash
dig app.provider.net A +short
```

## Fix

1. For the root domain set A and AAAA records with the provider's IPs.
2. If your DNS provider supports it: use ALIAS, ANAME or CNAME flattening instead of a CNAME.
3. Redirect the root domain to www via HTTP and create only www as CNAME.
4. On subdomains never mix a CNAME with other records of the same name.
5. Check the zone after changes and lower the TTL before switching.

## FAQ

**Why does a CNAME work on www but not on example.com?**

www has no other records, so a CNAME is allowed. The apex always needs SOA and NS, and nothing else may sit next to a CNAME.

**What does CNAME flattening do?**

The DNS provider resolves the target itself and serves A and AAAA records at the apex. To resolvers it looks like a normal A record.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dns-cname-at-zone-apex-not-allowed/](https://daniele-michelin.com/en/errors/dns-cname-at-zone-apex-not-allowed/)**
