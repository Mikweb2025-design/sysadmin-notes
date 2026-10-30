# PostgreSQL: FATAL: password authentication failed for user

> The server found a matching `pg_hba.conf` line that asks for a password, and the password was not accepted. The server log names the matching line and usually the reason.

Example:

```
psql: error: connection to server at "127.0.0.1", port 5432 failed: FATAL:  password authentication failed for user "appuser"
2026-10-12 09:14:03 CEST [4121] FATAL:  password authentication failed for user "appuser"
2026-10-12 09:14:03 CEST [4121] DETAIL:  Connection matched pg_hba.conf line 113: "host all all 127.0.0.1/32 scram-sha-256"
```

## Common causes

- The password is wrong, for example an old one in the app config, in an environment variable or in `~/.pgpass`.
- The role has no password set, or it was set in a different way than the line expects.
- Another `pg_hba.conf` line matches first. The first match wins, so a different user, database, address or method may apply.
- Special characters in the password break a connection URL, such as `@`, `:`, `/` or `%`, and must be percent-encoded.
- The user name has other spelling or case than the role in the database.

## How to check

Which roles exist and can log in?:

```bash
sudo -u postgres psql -c '\du'
```

Which line did the failed login match? Read the DETAIL in the log:

```bash
sudo tail -n 30 /var/log/postgresql/postgresql-*-main.log
```

Active rules in order:

```bash
sudo -u postgres psql -c 'SELECT line_number, type, database, user_name, address, auth_method FROM pg_hba_file_rules'
```

Password encryption in use:

```bash
sudo -u postgres psql -c 'SHOW password_encryption'
```

## Fix

1. Set a new password interactively so it does not land in the shell history: `sudo -u postgres psql`, then `\password appuser`.
2. Use the same password in the app config. Percent-encode special characters in a URL or switch to separate host, user and password parameters.
3. Fix the order and content of the `pg_hba.conf` lines, then reload: `sudo systemctl reload postgresql`. Never use `trust` for network addresses as a quick fix.
4. If an old client library supports only md5, update the library rather than lowering the security of the server. After changing `password_encryption`, set the passwords again.
5. Remove an outdated `~/.pgpass` entry or `PGPASSWORD` variable that overrides the password you think you use.

## FAQ

**Why does psql work on the server but not from the app?**

Locally, the socket often uses another rule (`peer`) than the network (`scram-sha-256`). Test with `psql -h 127.0.0.1` to use the same path as the app.

**How is this different from no pg_hba.conf entry?**

Here a line matched and the password failed. With “no pg_hba.conf entry” no line matched at all, so the server rejects before asking for a password.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postgresql-fatal-password-authentication-failed/](https://daniele-michelin.com/en/errors/postgresql-fatal-password-authentication-failed/)**
