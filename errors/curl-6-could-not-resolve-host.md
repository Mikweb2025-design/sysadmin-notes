# curl: (6) Could not resolve host

> curl could not turn the host name into an IP address, so it never tried to connect. The question is whether the name is wrong, the DNS record is missing, or only this machine cannot resolve it.

Example:

```
$ curl -I https://shop.example.com
curl: (6) Could not resolve host: shop.example.com
```

## Common causes

- The host name is misspelled, or the record does not exist yet, for example after a new domain, a new subdomain or a DNS migration.
- This machine uses a resolver that fails or has stale data, while others see the record.
- The record exists only in a different zone or in internal DNS, not in public DNS.
- `/etc/hosts` or a search domain overrides or changes the name.
- The server has no working DNS at all (see “Temporary failure in name resolution”).

## How to check

What does the system resolver say? This is what curl uses:

```bash
getent hosts shop.example.com
```

What does public DNS say?:

```bash
dig shop.example.com +short @1.1.1.1
```

Do the authoritative servers have the record?:

```bash
dig shop.example.com +trace +short
```

Is there an override in hosts?:

```bash
grep -n 'example' /etc/hosts
```

## Fix

1. Check the spelling of the host name in the command, the config or the URL.
2. If public DNS shows no record, add it at your DNS provider and wait for the TTL of a negative answer to pass.
3. If public DNS has the record but only this machine fails, fix the local resolver (`resolvectl status`) or flush its cache: `sudo resolvectl flush-caches`.
4. If it works with an IP, test with `curl --resolve shop.example.com:443:203.0.113.10 https://shop.example.com` to separate DNS from the web server.
5. Remove wrong lines from `/etc/hosts` and check the search domain in `resolvectl status`.

## FAQ

**dig works but curl does not. Why?**

`dig` asks DNS directly, `curl` uses the system resolver, which also reads `/etc/hosts` and uses its own cache. Compare with `getent hosts`.

**How long until a new record works everywhere?**

It depends on the TTL and on cached negative answers. See the TTL tool and the post about DNS propagation.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/curl-6-could-not-resolve-host/](https://daniele-michelin.com/en/errors/curl-6-could-not-resolve-host/)**
