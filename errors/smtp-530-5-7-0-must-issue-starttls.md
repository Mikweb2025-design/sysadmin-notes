# SMTP 530 5.7.0: must issue a STARTTLS command first

> On this port the server accepts neither AUTH nor MAIL FROM over an unencrypted connection. The client has to send STARTTLS first or use implicit TLS on port 465.

Example:

```
530 5.7.0 Must issue a STARTTLS command first
```

## Common causes

- Client or application set to “no encryption”.
- Port 587 configured with implicit TLS or port 465 with STARTTLS – port and mode do not match.
- Postfix enforces TLS via `smtpd_tls_security_level = encrypt` or `smtpd_tls_auth_only = yes`.
- The server's TLS certificate is missing or unreadable, so STARTTLS is not offered.

## How to check

Check that STARTTLS is offered and which certificate is sent:

```bash
openssl s_client -starttls smtp -connect mail.example.com:587 -servername mail.example.com </dev/null | head -n 5
```

Show TLS enforcement in Postfix:

```bash
postconf -n | grep -E 'smtpd_tls_security_level|smtpd_tls_auth_only|smtpd_tls_cert_file'
```

Show settings for submission (587) and smtps (465):

```bash
postconf -M submission/inet smtps/inet
```

Search the log for STARTTLS errors:

```bash
journalctl -u postfix --since '1 hour ago' | grep -i 'starttls'
```

## Fix

1. In the client choose port 587 with STARTTLS or port 465 with SSL/TLS – never 587 with “none”.
2. Set the TLS option in applications, e.g. PHPMailer `SMTPSecure = 'tls'` for 587 or `'ssl'` for 465.
3. On the server configure a valid certificate in `smtpd_tls_cert_file` and `smtpd_tls_key_file`, then run `postfix reload`.
4. Keep `smtpd_tls_security_level = may` on port 25 between servers, otherwise you reject mail from servers without TLS.

## FAQ

**Should I enforce TLS on port 25?**

Not globally. Port 25 receives mail from other servers and some of them do not speak TLS. Enforce TLS on 587 and 465 only.

**Why does 530 appear right after moving to a new mail server?**

The new server enforces TLS while the old one allowed plain logins. Printers, scanners and old scripts must be switched to STARTTLS.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-530-5-7-0-must-issue-starttls/](https://daniele-michelin.com/en/errors/smtp-530-5-7-0-must-issue-starttls/)**
