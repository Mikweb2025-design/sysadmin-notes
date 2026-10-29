# ERR_SSL_VERSION_OR_CIPHER_MISMATCH (SSL_ERROR_NO_CYPHER_OVERLAP)

> The browser and the server could not agree on a TLS version or cipher. Either the server offers only outdated protocols, or its settings are so strict that modern clients find no match.

Example:

```
This site can’t provide a secure connection
example.com uses an unsupported protocol.
ERR_SSL_VERSION_OR_CIPHER_MISMATCH
```

## Common causes

- The server allows only old protocols such as SSLv3, TLS 1.0 or 1.1, which current browsers refuse.
- `ssl_protocols` or `ssl_ciphers` are too narrow or contain a typo.
- The virtual host answering the request has no certificate or only one type the client does not support.
- A very old client, or an old OpenSSL on the server, cannot do TLS 1.2.
- HTTPS is requested on a port where the server speaks plain http.

## How to check

Which protocol and cipher does the server really offer?:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | grep -E 'Protocol|Cipher'
```

Does TLS 1.2 work?:

```bash
openssl s_client -connect example.com:443 -tls1_2 </dev/null 2>&1 | head -n 12
```

What is configured in nginx?:

```bash
nginx -T 2>/dev/null | grep -nE 'ssl_protocols|ssl_ciphers|ssl_certificate '
```

## Fix

1. Allow modern protocols: `ssl_protocols TLSv1.2 TLSv1.3;`.
2. Use a current cipher list, for example the Mozilla "intermediate" profile, or leave `ssl_ciphers` out and use the defaults.
3. Make sure the right server block answers: it needs `listen 443 ssl;` and a matching `ssl_certificate` and key.
4. Update OpenSSL and nginx on old servers.
5. Reload and test: `nginx -t && systemctl reload nginx`.

## FAQ

**Should I re-enable TLS 1.0 for an old client?**

No. TLS 1.0 and 1.1 are deprecated and unsafe. Update the client, or put a modern proxy in front of the old system.

**It works in one browser and not in another. Why?**

Browsers support different protocol and cipher sets. A very narrow server configuration may suit only some of them.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/tls-err-ssl-version-or-cipher-mismatch/](https://daniele-michelin.com/en/errors/tls-err-ssl-version-or-cipher-mismatch/)**
