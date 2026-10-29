# SSH: WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED

> The server presents a different host key than the one stored in your `known_hosts`. Often harmless (a reinstall), but it is the exact warning that would also appear in a man-in-the-middle attack.

Example:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Host key verification failed.
```

## Common causes

- The server was reinstalled or its SSH keys were regenerated.
- The IP address or hostname now belongs to a different machine, for example after a cloud rebuild.
- A load balancer or a cluster hands out different servers with different keys.
- DNS points to a different host than before.
- Rarely: someone is intercepting the connection.

## How to check

Which key is stored for this host?:

```bash
ssh-keygen -F server.example.com
```

Fingerprint the server shows now:

```bash
ssh-keyscan -t ed25519 server.example.com 2>/dev/null | ssh-keygen -lf -
```

Fingerprint on the server itself (use the console or a trusted channel):

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

## Fix

1. Compare the two fingerprints. If they match the new server you expect, continue.
2. Remove only the old entry: `ssh-keygen -R server.example.com` (and the IP address if it is listed too).
3. Connect again and accept the new key after checking its fingerprint.
4. If the key changed and you did not expect it, do not connect. Ask the owner or use the provider's console.
5. For rebuilt servers that should keep their identity, restore the old `/etc/ssh/ssh_host_*` files from backup.

## FAQ

**Can I turn the warning off with StrictHostKeyChecking=no?**

Technically yes, but it removes the protection against interception. Fix the entry instead, and never use that option permanently.

**Why does it appear after every rebuild of my cloud server?**

A new server generates new host keys. Automate the cleanup with `ssh-keygen -R` in your rebuild script, or keep and restore the host keys.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssh-remote-host-identification-has-changed/](https://daniele-michelin.com/en/errors/ssh-remote-host-identification-has-changed/)**
