# SSH: Received disconnect: Too many authentication failures

> The server counts every key your SSH agent offers as a failed attempt. With many keys loaded it reaches `MaxAuthTries` (6 by default) before your password or the right key is even tried.

Example:

```
Received disconnect from 203.0.113.10 port 22:2: Too many authentication failures
Disconnected from 203.0.113.10 port 22
user@203.0.113.10: Permission denied (publickey).
```

## Common causes

- The ssh-agent holds many keys and offers them one after another.
- No `IdentityFile` is set for the host, so ssh tries all default keys and agent keys.
- The right key is not authorized on the server.
- `MaxAuthTries` on the server is low.
- Old keys in `~/.ssh` are tried first.

## How to check

How many keys does the agent hold?:

```bash
ssh-add -l
```

Which keys are offered, and in which order?:

```bash
ssh -v user@server.example.com 2>&1 | grep -iE 'offering|authentications that can continue'
```

The limit on the server (console or another access):

```bash
sshd -T | grep -i maxauthtries
```

## Fix

1. Offer only the key you name: `ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 user@server.example.com`.
2. Make it permanent in `~/.ssh/config`: `Host server`, `IdentityFile ~/.ssh/id_ed25519`, `IdentitiesOnly yes`.
3. Remove unneeded keys from the agent: `ssh-add -D`, then add only what you need with `ssh-add ~/.ssh/id_ed25519`.
4. Check that the public key is in `~/.ssh/authorized_keys` on the server with the right permissions.
5. Raise `MaxAuthTries` on the server only if you really must; fewer keys is the better fix.

## FAQ

**Why can I not even type my password?**

Key attempts come first. The limit is reached before ssh gets to password authentication.

**Is this the same as a ban by fail2ban?**

No. This disconnect is a single connection hitting the per-connection limit. A fail2ban ban blocks your address and shows as a timeout or refusal, see [port 22 timed out](https://daniele-michelin.com/en/errors/ssh-connect-to-host-port-22-connection-timed-out/).

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssh-too-many-authentication-failures/](https://daniele-michelin.com/en/errors/ssh-too-many-authentication-failures/)**
