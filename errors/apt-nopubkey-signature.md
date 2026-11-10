# apt: NO_PUBKEY – The following signatures couldn't be verified

> apt does not know the public key that signed the repository. Until the key is present, packages from that source are not installed.

Example:

```
W: GPG error: https://repo.example.com/debian stable InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 1A2B3C4D5E6F7A8B
```

## Common causes

- The repo key was never imported, typical after adding a new source.
- The key expired and the vendor rotated it.
- The keyring is not at the expected path, often after a distribution upgrade.
- An old source points to a repository that is signed differently now.
- The system clock is wrong, so valid signatures look expired.

## How to check

Show the message and key ID:

```bash
apt-get update 2>&1 | grep -B1 -A3 -i 'NO_PUBKEY'
```

Existing keyrings:

```bash
ls -l /etc/apt/keyrings/ /usr/share/keyrings/ 2>/dev/null | head -n 20
```

Check the system clock:

```bash
date -u; timedatectl status | head -n 5
```

Show the affected source:

```bash
grep -rn . /etc/apt/sources.list.d/ | head
```

## Fix

1. Fetch the key from the repository vendor, not from a random keyserver: `curl -fsSL URL | gpg --dearmor -o /etc/apt/keyrings/repo.gpg`.
2. Point the source at the keyring: `signed-by=/etc/apt/keyrings/repo.gpg`.
3. For an expired key, check whether the vendor published a new one and update the source accordingly.
4. `--allow-unauthenticated` or disabling signature checks is not a fix, it is a security hole.
5. Run `apt-get update` again and read the full output.

## FAQ

**Can I ignore the warning?**

No. Without a valid signature apt can install manipulated packages. Either get the key or remove the source.

**Why does it appear after an upgrade?**

Because new key paths and keyrings are used and old sources do not bring their key along.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/apt-nopubkey-signature/](https://daniele-michelin.com/en/errors/apt-nopubkey-signature/)**
