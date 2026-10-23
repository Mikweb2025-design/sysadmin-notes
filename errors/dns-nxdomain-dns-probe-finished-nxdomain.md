# DNS NXDOMAIN: domain or hostname does not exist

> The responsible name server answered authoritatively that this name does not exist. That differs from SERVFAIL or a timeout – the question is which server says so and why.

Example:

```
DNS_PROBE_FINISHED_NXDOMAIN – ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 41382
```

## Common causes

- Typo in the hostname, or the record was created in the wrong zone (e.g. www.example.com.example.com).
- The record has not been created yet or was forgotten during a move.
- Domain expired, or name servers not (or no longer) correctly set at the registry.
- Negative cache: a resolver stored the NXDOMAIN; the SOA minimum value sets how long.
- Local causes: /etc/hosts, VPN with split DNS, Pi-hole or a corporate resolver blocking it.

## How to check

Show status and authority section of the answer:

```bash
dig www.example.com A +noall +comments +answer +authority
```

Check the delegation at the TLD:

```bash
dig example.com NS +short @$(dig +short NS com. | head -n 1)
```

Ask the authoritative name server directly:

```bash
dig www.example.com A +short @$(dig +short NS example.com | head -n 1)
```

Check domain status and expiry:

```bash
whois example.com | grep -iE 'status|expir|name server'
```

## Fix

1. Use the authoritative server's answer as reference – if it returns the record, it is a caching issue.
2. Create the missing record; in zone editors usually enter just `www`, not the full name.
3. Renew an expired domain at the registrar and check the NS records.
4. With negative caching wait (SOA minimum) or flush local caches: `resolvectl flush-caches`.
5. Before a move export all records and test them against the new name server afterwards.

## FAQ

**How long does an NXDOMAIN stay cached?**

As long as the SOA minimum field or SOA TTL says, often 300 to 3600 seconds. Only then does the resolver ask again.

**How is it different from SERVFAIL?**

NXDOMAIN means the name definitely does not exist. SERVFAIL means the resolver could not get a valid answer, e.g. due to DNSSEC or unreachable name servers.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dns-nxdomain-dns-probe-finished-nxdomain/](https://daniele-michelin.com/en/errors/dns-nxdomain-dns-probe-finished-nxdomain/)**
