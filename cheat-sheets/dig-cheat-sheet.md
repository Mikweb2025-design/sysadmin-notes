# dig cheat sheet

The dig queries I actually use for DNS migrations and mail issues – searchable and copyable in one click.

## Basic lookups

- Query the IPv4 address (A record)

  ```bash
  dig example.com A
  ```
- Query the IPv6 address (AAAA record)

  ```bash
  dig example.com AAAA
  ```
- Show mail servers with priority

  ```bash
  dig example.com MX +short
  ```
- All TXT records (SPF, verifications)

  ```bash
  dig example.com TXT +short
  ```
- Nameservers responsible for the zone

  ```bash
  dig example.com NS +short
  ```
- SOA with serial number (check zone changes)

  ```bash
  dig example.com SOA +short
  ```
- CAA: which CAs may issue certificates

  ```bash
  dig example.com CAA +short
  ```
- Show the CNAME target of a hostname

  ```bash
  dig www.example.com CNAME +short
  ```
- Print only the answer section incl. TTL

  ```bash
  dig +noall +answer example.com
  ```

## Resolvers & nameservers

- Query through a specific resolver (Cloudflare)

  ```bash
  dig @1.1.1.1 example.com A +short
  ```
- Ask the authoritative nameserver directly (no cache)

  ```bash
  dig @ns1.example.com example.com A +norecurse
  ```
- Follow the delegation from the root zone

  ```bash
  dig +trace example.com
  ```
- Compare answers from several public resolvers

  ```bash
  for r in 1.1.1.1 8.8.8.8 9.9.9.9; do echo "$r: $(dig +short @$r example.com A)"; done
  ```
- Check the remaining TTL in the resolver cache

  ```bash
  dig @1.1.1.1 +noall +answer example.com A
  ```
- Force the query over TCP instead of UDP

  ```bash
  dig +tcp example.com A
  ```
- Send the query over IPv6 transport

  ```bash
  dig -6 @2606:4700:4700::1111 example.com A
  ```

## Mail: SPF, DKIM, DMARC, PTR

- Filter out the SPF record

  ```bash
  dig example.com TXT +short | grep spf1
  ```
- Query the DMARC policy

  ```bash
  dig _dmarc.example.com TXT +short
  ```
- Query the DKIM public key for a selector

  ```bash
  dig SELECTOR._domainkey.example.com TXT +short
  ```
- Reverse lookup (PTR) of an IPv4 address

  ```bash
  dig -x 203.0.113.10 +short
  ```
- Reverse lookup (PTR) of an IPv6 address

  ```bash
  dig -x 2001:db8::1 +short
  ```
- Query an SRV record (e.g. mail autoconfig)

  ```bash
  dig _submission._tcp.example.com SRV +short
  ```

## DNSSEC

- Show the answer with RRSIG signatures

  ```bash
  dig example.com A +dnssec +multi
  ```
- Query the zone's DNSKEY records

  ```bash
  dig example.com DNSKEY +short
  ```
- Check the DS record in the parent zone

  ```bash
  dig example.com DS +short
  ```
- Check validation: ad flag = DNSSEC valid

  ```bash
  dig @1.1.1.1 example.com A +dnssec | grep flags
  ```
- Query with validation disabled (isolate SERVFAIL)

  ```bash
  dig @1.1.1.1 example.com A +cd
  ```

## Zones & advanced

- Attempt a zone transfer (should be REFUSED for outsiders) (**careful**)

  ```bash
  dig @ns1.example.com example.com AXFR
  ```
- Show the nameserver ID (identify anycast node)

  ```bash
  dig @ns1.example.com example.com SOA +nsid
  ```
- Run queries in batch from a file

  ```bash
  dig -f hosts.txt +noall +answer
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/dig-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/dig-cheat-sheet/)**
