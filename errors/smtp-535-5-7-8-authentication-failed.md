# SMTP 535 5.7.8: authentication failed – login rejected

> The server rejected the SMTP AUTH login. The connection works, but username, password or auth mechanism do not match what the backend (usually Dovecot SASL) expects.

Example:

```
535 5.7.8 Error: authentication failed: authentication failure
```

## Common causes

- Wrong password, or the password changed and the client still uses the old one.
- Username in the wrong format: full address instead of local part or the other way round (`auth_username_format`).
- Gmail/Google Workspace with 2FA: regular password instead of an app password.
- Microsoft 365: SMTP AUTH disabled for the mailbox (there usually shown as 535 5.7.139).
- Dovecot auth socket for Postfix missing or with wrong permissions.

## How to check

Show SASL errors from Postfix and Dovecot:

```bash
journalctl -u postfix -u dovecot --since '30 min ago' | grep -iE 'sasl|auth'
```

Test the login directly against Dovecot (prompts for the password):

```bash
doveadm auth test -x service=smtp info@example.com
```

Check how Postfix hands auth to SASL:

```bash
postconf smtpd_sasl_type smtpd_sasl_path smtpd_sasl_auth_enable
```

Show auth mechanisms and socket in Dovecot:

```bash
doveconf -n | grep -E 'auth_mechanisms|auth_username_format|unix_listener'
```

## Fix

1. Re-enter the password in the client and verify it with `doveadm auth test`.
2. Use the username exactly as the passdb expects it – usually the full address.
3. For Google create an app password; for Microsoft 365 enable SMTP AUTH per mailbox or move to OAuth.
4. For Postfix set `auth_mechanisms = plain login` and create the socket `/var/spool/postfix/private/auth` with `user = postfix`.
5. Many 535s from one IP are brute force: enable fail2ban for Postfix SASL.

## FAQ

**What is the difference between 535 5.7.8 and 535 5.7.3?**

Both report a failed login. 5.7.8 means invalid credentials, 5.7.3 is used by Exchange when authentication is not allowed. The checks are the same.

**Why does IMAP work but SMTP fails?**

IMAP talks to Dovecot directly, SMTP goes through Postfix and the SASL socket. If the socket is missing or the mechanism differs, only sending breaks.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smtp-535-5-7-8-authentication-failed/](https://daniele-michelin.com/en/errors/smtp-535-5-7-8-authentication-failed/)**
