# DNS SERVFAIL: find the cause and fix it

> The resolver could not produce a valid answer – either the authoritative nameservers are not answering correctly or DNSSEC validation fails.

Example:

```
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 41234
```

## Common causes

- The DS record at the registry no longer matches the zone's DNSKEY after a provider change.
- Expired RRSIG signatures because automatic re-signing stopped.
- Lame delegation: listed nameservers are not authoritative for the zone or unreachable.
- Nameservers answer incorrectly or block TCP/EDNS (large responses).

## How to check

Query with DNSSEC data:

```bash
dig example.com A +dnssec
```

Same query with checking disabled – if it works, DNSSEC is the problem:

```bash
dig example.com A +cd
```

Query the DS record at the parent zone:

```bash
dig +short DS example.com @a.gtld-servers.net
```

Query DNSKEY directly at the authoritative server:

```bash
dig +short DNSKEY example.com @ns1.example-dns.com
```

Show the validation chain with a reason:

```bash
delv example.com A
```

## Fix

1. Delete the wrong DS record at the registrar or replace it with the matching one; mind the DS TTL.
2. When moving a DNSSEC-signed domain, remove the DS first, wait out the TTL, then move and re-sign.
3. Check and fix automatic re-signing at the DNS provider or in BIND/PowerDNS.
4. Align NS records at the registry and in the zone.
5. Flush resolver caches after the fix, e.g. `unbound-control flush_zone example.com`.

## FAQ

**Why does it work for some users but not others?**

Only validating resolvers (e.g. 1.1.1.1, 8.8.8.8, many ISPs) return SERVFAIL. Non-validating resolvers still hand out the answer.

**How long after the fix until it works?**

As long as the TTL of the DS record or of cached negative answers – often one to 24 hours.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dns-servfail-dnssec/](https://daniele-michelin.com/en/errors/dns-servfail-dnssec/)**
