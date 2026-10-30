# nginx: [emerg] cannot load certificate (PEM no start line, key values mismatch)

> nginx could not read the certificate or the key it was told to use, so it refuses to start or reload. The file is missing or unreadable, it is not PEM, it is the wrong file, or the key does not belong to the certificate.

Example:

```
nginx: [emerg] cannot load certificate "/etc/nginx/ssl/example.com.crt": BIO_new_file() failed (SSL: error:80000002:system library::No such file or directory)
nginx: [emerg] cannot load certificate key "/etc/nginx/ssl/example.com.key": PEM_read_bio_PrivateKey() failed (SSL: error:...:PEM routines::no start line)
```

## Common causes

- The path in `ssl_certificate` or `ssl_certificate_key` is wrong, or the file does not exist yet.
- The files are swapped: the key is in `ssl_certificate` or the certificate is in `ssl_certificate_key`.
- The certificate does not match the private key (message `key values mismatch`), typical after a renewal with a new key.
- The file is not PEM: DER or PKCS#12 format, empty, or damaged by copy and paste (missing lines, no newline between concatenated certificates).
- The file cannot be read at nginx start because of permissions.

## How to check

Which exact error does nginx report?:

```bash
nginx -t
```

Read the certificate (subject and dates):

```bash
openssl x509 -in /etc/nginx/ssl/example.com.crt -noout -subject -enddate
```

Do certificate and key belong together? (both hashes must be identical):

```bash
openssl x509 -in /etc/nginx/ssl/example.com.crt -noout -pubkey | openssl sha256; sudo openssl pkey -in /etc/nginx/ssl/example.com.key -pubout | openssl sha256
```

Does the file start and end like a PEM file?:

```bash
sudo head -n 1 /etc/nginx/ssl/example.com.crt; sudo tail -n 1 /etc/nginx/ssl/example.com.crt
```

## Fix

1. Correct the paths in the server block. With Let's Encrypt use `/etc/letsencrypt/live/example.com/fullchain.pem` and `privkey.pem`.
2. Use the full chain file (certificate first, then intermediates) in `ssl_certificate` and the private key only in `ssl_certificate_key`.
3. If the hashes differ, use the key that was generated together with the certificate, or issue the certificate again for the current key.
4. Convert non-PEM files, for example `openssl x509 -inform der -in cert.der -out cert.pem`, and make sure each block ends with a newline.
5. Test and apply: `nginx -t && systemctl reload nginx`.

## FAQ

**Does nginx start if only the chain is incomplete?**

Yes, but some clients will show a certificate error. The check in the TLS chain post shows whether intermediates are missing.

**Why is the error only shown at reload and not before?**

nginx reads the files when the configuration is loaded. A reload with a new, broken file fails the test, and the running instance keeps the old certificate.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-emerg-cannot-load-certificate-pem-no-start-line/](https://daniele-michelin.com/en/errors/nginx-emerg-cannot-load-certificate-pem-no-start-line/)**
