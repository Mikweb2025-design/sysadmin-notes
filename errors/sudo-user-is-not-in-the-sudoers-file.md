# sudo: user is not in the sudoers file. This incident will be reported.

> sudo works, but the policy does not allow this user. On Debian and Ubuntu the usual rule is: members of the group `sudo` may use it. The user is not in that group, or the group change is not active yet.

Example:

```
$ sudo apt update
[sudo] password for anna:
anna is not in the sudoers file.  This incident will be reported.
```

## Common causes

- The user is not a member of the `sudo` group. On Debian the first user is often not added when a root password was set during the installation.
- The user was added to the group, but the current session still has the old group list. A new login is needed.
- The line `%sudo ALL=(ALL:ALL) ALL` was removed or changed in `/etc/sudoers` or in a file in `/etc/sudoers.d/`.
- A user name rule uses a different spelling, for example another domain name or case.

## How to check

Which groups does the user have in this session?:

```bash
id
```

Who is in the sudo group?:

```bash
getent group sudo
```

Which sudo rules exist? (needs root, use su):

```bash
su -c 'grep -rvE "^(#|$)" /etc/sudoers /etc/sudoers.d/'
```

Is the sudoers syntax valid?:

```bash
su -c 'visudo -c'
```

## Fix

1. Become root with `su -` (needs the root password) or use another admin account or the console.
2. Add the user: `usermod -aG sudo anna`. Keep the `-a`, otherwise the user loses all other groups.
3. Log out and in again, or start a new SSH session, then check with `id`.
4. To change rules, edit only with `visudo` or `visudo -f /etc/sudoers.d/name`. It checks the syntax before saving. A broken sudoers file can lock you out of sudo.
5. If sudo is not installed at all, install it as root: `apt install sudo`.

## FAQ

**Who sees “This incident will be reported”?**

It is written to the system log, usually the authentication log or the journal. It is not sent to anyone automatically. An admin can find it there.

**Why does the group change not work right away?**

Group membership is read at login. An open terminal keeps the old list until you log in again.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/sudo-user-is-not-in-the-sudoers-file/](https://daniele-michelin.com/en/errors/sudo-user-is-not-in-the-sudoers-file/)**
