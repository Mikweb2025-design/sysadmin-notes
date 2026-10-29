# Postfix: SASL authentication failed; cannot authenticate to server (no mechanism available)

> Postfix tried to log in at your smart host (relay) and failed. This is the outgoing side: your server is the client. The cause is a missing SASL module, wrong credentials, or the login being offered only after TLS starts.

Example:

```
postfix/smtp[2314]: warning: SASL authentication failure: No worthy mechs found
postfix/smtp[2314]: 4F1A2B3C4D: SASL authentication failed; cannot authenticate to server smtp.example.com[203.0.113.10]: no mechanism available
```

## Common causes

- The package with the SASL mechanisms is missing: `libsasl2-modules`. That gives “no mechanism available”.
- The user name or password in `/etc/postfix/sasl_passwd` is wrong, or the file was changed but `postmap` was not run.
- The key in `sasl_passwd` does not match `relayhost` exactly, for example the square brackets or the port.
- The provider offers login only over TLS, but `smtp_tls_security_level` is not set.
- `smtp_sasl_security_options` forbids the mechanism the provider offers.

## How to check

Is the SASL module package installed?:

```bash
dpkg -l | grep libsasl2-modules
```

Relay, SASL and TLS settings:

```bash
postconf -n | grep -E 'relayhost|smtp_sasl|smtp_tls'
```

Does the stored key match the relay host?:

```bash
sudo postmap -q '[smtp.example.com]:587' /etc/postfix/sasl_passwd
```

What does the log say?:

```bash
journalctl -u postfix --since '1 hour ago' --no-pager | grep -i sasl | tail -n 10
```

## Fix

1. Install the modules: `apt install libsasl2-modules`.
2. Set the relay with brackets and the port: `relayhost = [smtp.example.com]:587`.
3. Enable login and TLS: `smtp_sasl_auth_enable = yes`, `smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd`, `smtp_sasl_security_options = noanonymous`, `smtp_tls_security_level = encrypt`.
4. Put `[smtp.example.com]:587 user:password` in `sasl_passwd`, then `postmap /etc/postfix/sasl_passwd` and `chmod 600 /etc/postfix/sasl_passwd*`.
5. Reload: `postfix reload`, and test with a mail.

## FAQ

**What does “no mechanism available” mean?**

Postfix has no usable SASL mechanism for this server. Either the plugin package is missing, or the server offers its mechanisms only after STARTTLS, which Postfix did not start.

**How is this different from error 535?**

With [535](https://daniele-michelin.com/en/errors/smtp-535-5-7-8-authentication-failed/) the server answered and refused the credentials. Here Postfix could not even try.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postfix-sasl-authentication-failed-no-mechanism-available/](https://daniele-michelin.com/en/errors/postfix-sasl-authentication-failed-no-mechanism-available/)**
