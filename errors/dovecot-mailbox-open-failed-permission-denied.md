# Dovecot: Error: open() failed: Permission denied (mailbox)

> The user Dovecot runs as for this mailbox may not read or write the mail files. Typical after a migration, a new server, or when Postfix delivers as a different user than Dovecot reads.

Example:

```
dovecot: imap(user@example.com): Error: open(/var/mail/user) failed: Permission denied (euid=1000(user) egid=1000(user) missing +w perm: /var/mail, dir owned by 0:8 mode=0775)
```

## Common causes

- The mail files belong to another user than the one Dovecot uses, for example after copying as root.
- `mail_location` points to the wrong path, or the directory does not exist.
- `mail_uid`, `mail_gid` or `first_valid_uid` do not match the real owner.
- The system mailbox `/var/mail` needs the mail group, which Dovecot does not have.
- SELinux or AppArmor blocks access to the mail directory.

## How to check

Which location and user settings are active?:

```bash
doveconf -n | grep -E 'mail_location|mail_uid|mail_gid|first_valid_uid'
```

Who owns the mail directory?:

```bash
ls -ld /var/mail /var/vmail
```

What does Dovecot think for this account?:

```bash
doveadm user user@example.com
```

The exact errors:

```bash
journalctl -u dovecot --since '1 hour ago' --no-pager | grep -i 'permission denied' | tail -n 10
```

## Fix

1. Set the right location, for example `mail_location = maildir:/var/vmail/%d/%n` or `maildir:~/Maildir`.
2. Fix the owner: `chown -R vmail:vmail /var/vmail` and `chmod -R u+rwX,g-rwx,o-rwx /var/vmail`.
3. Make `mail_uid` and `mail_gid` match that owner, and `first_valid_uid` lower than it.
4. For `/var/mail`, set `mail_privileged_group = mail` so Dovecot can create lock files.
5. Reload: `doveadm reload` and test the login again.

## FAQ

**Why does it work for new mail but not for old mail?**

Old files were created by another user or tool. Fix the owner for the whole mail tree, not only for new folders.

**Do Postfix and Dovecot have to use the same user?**

For local delivery, the user who writes the files and the user who reads them must be able to access the same files. A virtual mail user such as `vmail` for both is the simplest setup.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dovecot-mailbox-open-failed-permission-denied/](https://daniele-michelin.com/en/errors/dovecot-mailbox-open-failed-permission-denied/)**
