# SSL: unable to get local issuer certificate

> The client cannot build the chain from the server certificate to a trusted root CA. Either the server does not send the intermediates, or the client lacks the root CA.

Example:

```
curl: (60) SSL certificate problem: unable to get local issuer certificate
```

## Common causes

- The server sends only its own certificate, not the chain (`cert.pem` instead of `fullchain.pem`).
- The client's CA bundle is missing or outdated (package `ca-certificates`).
- Python on macOS without installed certificates, or a venv with its own `certifi`.
- A corporate proxy with TLS inspection replaces certificates with an internal CA.
- Internal or private CA installed only on some machines.

## How to check

Show the chain that is sent: subject and issuer of each certificate:

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts </dev/null 2>/dev/null | grep -E '^ *[0-9] s:|^ *i:'
```

Show curl error details:

```bash
curl -vI https://example.com 2>&1 | grep -iE 'issuer|verify|SSL certificate'
```

Show Python's CA paths:

```bash
python3 -c 'import ssl; print(ssl.get_default_verify_paths())'
```

Check that the system CA bundle exists:

```bash
openssl x509 -in /etc/ssl/certs/ca-certificates.crt -noout -subject 2>/dev/null | head -n 1
```

## Fix

1. On the server send the full chain: nginx `ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;`.
2. On the client update the CA bundle: `apt install --reinstall ca-certificates && update-ca-certificates`.
3. Add a private CA to the system: copy it to `/usr/local/share/ca-certificates/` and run `update-ca-certificates`.
4. With TLS inspection add the corporate CA to the client, e.g. via `REQUESTS_CA_BUNDLE` or `NODE_EXTRA_CA_CERTS`.
5. Never use `curl -k` or `verify=False` permanently – it makes TLS worthless.

## FAQ

**Why does it work in the browser but not with curl?**

Browsers often fetch missing intermediates themselves or have them cached. curl and Python do not, so they show the real problem.

**How do I spot a missing chain?**

`openssl s_client -showcerts` shows only one certificate (0 s:…). With a correct chain at least one intermediate follows (1 s:…).

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssl-unable-to-get-local-issuer-certificate/](https://daniele-michelin.com/en/errors/ssl-unable-to-get-local-issuer-certificate/)**
