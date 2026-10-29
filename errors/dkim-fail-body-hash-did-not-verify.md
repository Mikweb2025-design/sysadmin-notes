# DKIM: dkim=fail (body hash did not verify)

> The signature is intact, but the body of the message no longer matches the hash that was signed. Something changed the text after signing: a footer, a virus banner, a disclaimer, rewritten links or even changed line endings.

Example:

```
Authentication-Results: mx.google.com;
       dkim=fail (body hash did not verify) header.i=@example.com header.s=default header.b=abc123;
       spf=pass smtp.mailfrom=example.com
```

## Common causes

- A mail gateway or antivirus adds a footer or banner after your server signed the message.
- A mailing list or a forwarder changes the subject, the text or adds an unsubscribe footer.
- Link tracking or URL rewriting modifies the body.
- Strict canonicalization (`simple`) breaks on small changes such as whitespace or line endings.
- The message is signed before another filter, or signed twice with different keys.

## How to check

Look at the signature and the result in the headers of a received test mail:

```bash
grep -iE '^(dkim-signature|authentication-results)' message.eml
```

Is the DNS key published and valid?:

```bash
dig +short TXT default._domainkey.example.com
```

Test the key on the signing server:

```bash
opendkim-testkey -d example.com -s default -vvv
```

Which canonicalization is configured?:

```bash
grep -i Canonicalization /etc/opendkim.conf
```

## Fix

1. Make signing the very last step, after antivirus, disclaimer and filter, just before sending.
2. Use relaxed canonicalization: `Canonicalization relaxed/relaxed`.
3. Remove or move footers and banners that are added after signing, or add them before.
4. Turn off link rewriting for your own outgoing mail, or sign after it.
5. For mailing lists, rely on SPF alignment and a `p=none` or `quarantine` DMARC policy; lists change messages by design.

## FAQ

**What is the difference to “signature did not verify”?**

“Body hash did not verify” means the text changed. “Signature did not verify” usually means the DNS key does not match the private key, or the signed headers changed.

**Does a DKIM failure always mean spam?**

No. If SPF passes and is aligned, DMARC can still pass. But a broken DKIM weakens your delivery, so fix it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dkim-fail-body-hash-did-not-verify/](https://daniele-michelin.com/en/errors/dkim-fail-body-hash-did-not-verify/)**
