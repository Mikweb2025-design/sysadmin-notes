# NET::ERR_CERT_COMMON_NAME_INVALID

> The certificate is valid but not issued for this name. Browsers only check the Subject Alternative Names (SAN) – if the requested hostname is missing there, you get this error.

Example:

```
NET::ERR_CERT_COMMON_NAME_INVALID – This server could not prove that it is www.example.com; its security certificate is from example.com.
```

## Common causes

- Certificate covers example.com only, but www.example.com is requested (or the other way round).
- nginx serves another vhost's certificate because no `server_name` matches and the `default_server` answers.
- A new subdomain was added but the certificate was not expanded.
- Access via IP address or an internal hostname.
- DNS points to a different server, e.g. an old hoster or a CDN without a matching certificate.

## How to check

Show the names in the served certificate (SAN):

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName
```

Show existing Let's Encrypt certificates and their domains:

```bash
certbot certificates 2>/dev/null | grep -E 'Certificate Name|Domains'
```

Show server_name, certificate paths and default_server:

```bash
nginx -T 2>/dev/null | grep -nE '^\s*(server_name|ssl_certificate|listen .*default_server)'
```

Check that both names point to the same server:

```bash
dig +short A www.example.com; dig +short A example.com
```

## Fix

1. Expand the certificate with the missing name: `certbot --nginx --expand -d example.com -d www.example.com`.
2. Create a matching `server_name` with the right `ssl_certificate` for every name.
3. Set up a separate `default_server` with a dummy certificate and `return 444;` so wrong names never hit a customer site.
4. Check DNS if the certificate comes from a foreign server.
5. After changes run `nginx -t && systemctl reload nginx`.

## FAQ

**Is the common name (CN) in the certificate enough?**

No. Modern browsers ignore the CN and only check the SAN list. Self-made certificates therefore need a subjectAltName.

**Does a wildcard certificate cover the apex domain?**

No. *.example.com covers www.example.com, but not example.com itself and not a.b.example.com. The apex must be listed too.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/net-err-cert-common-name-invalid/](https://daniele-michelin.com/en/errors/net-err-cert-common-name-invalid/)**
