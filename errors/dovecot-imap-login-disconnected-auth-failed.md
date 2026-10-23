# Dovecot: imap-login Disconnected (auth failed)

> A client logged in to IMAP or POP3 and was refused. That can be a single user with a wrong password, a broken passdb or an attack.

Example:

```
imap-login: Disconnected (auth failed, 1 attempts in 2 secs): user=<anna@example.com>, method=PLAIN, rip=198.51.100.7, lip=203.0.113.10, TLS
```

## Common causes

- Wrong or changed password, old value still saved on the phone or in Outlook.
- Username with or without domain, matching `auth_username_format` but not the client.
- Plaintext login without TLS is blocked (“Plaintext authentication disallowed”).
- passdb query or password scheme (e.g. SHA512-CRYPT) does not match the stored hashes.
- Many attempts from foreign IPs: brute force against IMAP/POP3.

## How to check

Group failed attempts by IP:

```bash
journalctl -u dovecot --since '1 hour ago' | grep 'auth failed' | grep -o 'rip=[0-9a-f.:]*' | sort | uniq -c | sort -rn | head
```

Test the login directly in Dovecot (prompts for the password):

```bash
doveadm auth test -x service=imap anna@example.com
```

Show auth and SSL settings:

```bash
doveconf -n | grep -E 'auth_|passdb|userdb|ssl ='
```

Show recent errors kept by Dovecot:

```bash
doveadm log errors
```

## Fix

1. For details temporarily set `auth_verbose = yes` and `auth_debug_passwords = no`, then run `doveadm reload`.
2. Reset the password and generate the hash with `doveadm pw -s SHA512-CRYPT`.
3. Move clients to port 993 (IMAPS) or 143 with STARTTLS.
4. Against brute force enable fail2ban with the `dovecot` filter.
5. Turn `auth_verbose` off again after debugging so usernames do not pile up in the log.

## FAQ

**How do I tell a wrong password from an attack?**

One user from a known IP is usually an old saved password. Many different usernames from a few IPs is an attack.

**What does “no auth attempts” mean instead of “auth failed”?**

With “no auth attempts” the client never tried to log in – usually port scanners or monitoring. That is not a login error.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dovecot-imap-login-disconnected-auth-failed/](https://daniele-michelin.com/en/errors/dovecot-imap-login-disconnected-auth-failed/)**
