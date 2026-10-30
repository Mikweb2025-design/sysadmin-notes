# DNS: status: REFUSED (query refused)

> The server was reached and answered on purpose with “no”. It is a policy decision, not a timeout. Either it does not serve the zone you asked for, or it does not do recursion for your address.

Example:

```
$ dig example.com @ns1.example.net
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 41873
;; WARNING: recursion requested but not available
```

## Common causes

- The domain points (NS records at the registrar) to a name server that does not host the zone. This is a “lame delegation”, typical after a DNS migration or a cancelled hosting.
- You asked a recursive resolver from outside its allowed network, for example your office DNS from the internet.
- An access list (`allow-query`, `allow-recursion`) or a firewall policy blocks your address.
- The zone is not loaded or failed to load on the server, for example because of a syntax error.
- Transfers between name servers (AXFR) are refused. This is normal and intended for unknown hosts.

## How to check

Which name servers does the domain use?:

```bash
dig example.com NS +short
```

Does each of them answer for the zone? (replace the name):

```bash
dig @ns1.example.net example.com SOA +norecurse
```

The full delegation path:

```bash
dig +trace example.com
```

Does your own resolver answer?:

```bash
dig example.com A +short
```

## Fix

1. If an authoritative server answers REFUSED for your zone, create or load the zone there, or remove that server from the NS records at the registrar.
2. After a provider change update the NS records at the registrar to the new provider only, and keep the old zone active until the old TTL has passed.
3. If it is your own resolver, allow your clients, for example in BIND `allow-recursion { 192.0.2.0/24; };`. Do not open recursion to the whole internet, it is abused for amplification attacks.
4. On the server read the log for zone load errors (`journalctl -u named` or `-u bind9`) and run `named-checkzone` on the zone file.
5. Retest from the failing client with `dig` directly against the server.

## FAQ

**What is the difference between REFUSED, SERVFAIL and NXDOMAIN?**

NXDOMAIN: the name does not exist. SERVFAIL: the server tried and failed. REFUSED: the server will not answer you at all.

**Why does only one of my name servers refuse?**

Usually the zone exists on one but not on the other. Resolvers pick one name server per query, so the error appears only now and then.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dns-status-refused-query-refused/](https://daniele-michelin.com/en/errors/dns-status-refused-query-refused/)**
