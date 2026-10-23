# SSH: permission denied (publickey)

> The server only allows key login and accepted none of the keys offered. The cause shows up in the client with `-vvv` and in the server log.

Example:

```
admin@203.0.113.10: Permission denied (publickey).
```

## Common causes

- Wrong username, e.g. root instead of ubuntu or admin.
- Public key missing from `~/.ssh/authorized_keys`, or wrapped across lines there.
- Permissions too open: home group-writable, `.ssh` not 700 or `authorized_keys` not 600 (StrictModes).
- Old RSA key with SHA-1 signature: newer OpenSSH versions reject `ssh-rsa`.
- The agent offers too many keys and the server stops after `MaxAuthTries`.

## How to check

See in the client which keys are offered and rejected:

```bash
ssh -vvv admin@203.0.113.10 2>&1 | grep -E 'Offering|Server accepts|Authentications that can continue|no mutual'
```

Check the server log for the login:

```bash
journalctl -u ssh -u sshd --since '10 min ago' | grep -iE 'authentication|refused|bad ownership'
```

Check permissions on home, .ssh and authorized_keys:

```bash
ls -ld ~admin ~admin/.ssh ~admin/.ssh/authorized_keys
```

Show effective sshd settings:

```bash
sshd -T | grep -E 'pubkeyauthentication|authorizedkeysfile|allowusers|pubkeyacceptedalgorithms'
```

## Fix

1. Connect with the right key: `ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes admin@host`.
2. Install the key properly: `ssh-copy-id -i ~/.ssh/id_ed25519.pub admin@host` or via the hoster's console.
3. Set permissions: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys` and check ownership.
4. Replace old RSA keys with Ed25519 instead of re-enabling `ssh-rsa`.
5. With SELinux: `restorecon -Rv ~/.ssh`.

## FAQ

**How do I get in if I am locked out?**

Through the hoster's web or rescue console. Add the key to authorized_keys there and check permissions – do not enable PasswordAuthentication permanently.

**Why does one key work and another does not?**

The server only accepts keys whose public part is in authorized_keys and whose type is allowed. `ssh -vvv` shows which key was offered.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssh-permission-denied-publickey/](https://daniele-michelin.com/en/errors/ssh-permission-denied-publickey/)**
