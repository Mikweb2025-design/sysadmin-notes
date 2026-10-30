# NET::ERR_CERT_AUTHORITY_INVALID: certificate authority not trusted

> The browser got a certificate but cannot trace it back to a certificate authority it trusts. This is about who signed it, not about the date or the name.

Example:

```
Your connection is not private
Attackers might be trying to steal your information from cloud.example.com
NET::ERR_CERT_AUTHORITY_INVALID
```

## Common causes

- The certificate is self-signed, for example a default one from an appliance, router or fresh install.
- It was issued by an internal CA whose root certificate is not installed on the client.
- The server sends only the leaf certificate without the intermediate, and the client cannot fetch it.
- A proxy, antivirus or firewall re-signs HTTPS traffic with its own root that the browser does not know.
- The wrong virtual host answers, so a default certificate is served instead of yours.

## How to check

Who issued the certificate, and for whom?:

```bash
echo | openssl s_client -connect cloud.example.com:443 -servername cloud.example.com 2>/dev/null | openssl x509 -noout -subject -issuer
```

What does the chain look like?:

```bash
echo | openssl s_client -connect cloud.example.com:443 -servername cloud.example.com -showcerts 2>/dev/null | grep -E '^ *[0-9] s:|^ *i:'
```

What does curl verify say?:

```bash
curl -vI https://cloud.example.com 2>&1 | grep -iE 'issuer|verify|SSL certificate'
```

Does the SNI name select the right certificate?:

```bash
echo | openssl s_client -connect cloud.example.com:443 -servername wrong.example.com 2>/dev/null | openssl x509 -noout -subject
```

## Fix

1. If subject and issuer are identical, the certificate is self-signed. For a public host, replace it with one from a public CA such as Let's Encrypt.
2. For an internal CA, install the root on the clients. On Debian and Ubuntu: copy the `.crt` to `/usr/local/share/ca-certificates/` and run `sudo update-ca-certificates`. Browsers and phones need the root imported into their own store.
3. Serve the full chain: in nginx `ssl_certificate` should point to `fullchain.pem`, not `cert.pem`.
4. If a proxy re-signs traffic, trust its root on that device or exempt the site from inspection.
5. Check that the right `server_name` has the right certificate, then reload the web server.

## FAQ

**Is it safe to click through the warning?**

Only for a host you control and recognise, on a network you trust. For anything else the warning exists to stop a man-in-the-middle, so do not accept it.

**Why does it work in one browser and fail in another?**

Browsers and operating systems keep their own trust stores, and some fetch missing intermediates while others do not. A missing intermediate is a common cause when only some clients fail.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/net-err-cert-authority-invalid-self-signed/](https://daniele-michelin.com/en/errors/net-err-cert-authority-invalid-self-signed/)**
