# MySQL / MariaDB: ERROR 1045 (28000) access denied for user

> The server was reached, but it rejects the login: wrong password, a user that does not exist for this host, or an authentication plugin that does not allow a password.

Example:

```
ERROR 1045 (28000): Access denied for user 'app'@'localhost' (using password: YES)
```

## Common causes

- The password is wrong or was changed, or it contains characters the shell or a config file mangles.
- The user exists only for another host: `'app'@'%'` is not the same as `'app'@'localhost'`.
- On MariaDB the `root` account uses the `unix_socket` plugin: it works with `sudo mysql` but not with a password.
- The application config still holds the old credentials after a migration or restore.
- Reverse DNS or `skip-name-resolve` changes which host entry matches.

## How to check

Which users and hosts exist, and with which plugin?:

```bash
sudo mysql -e "SELECT user, host, plugin FROM mysql.user;"
```

What can this user do?:

```bash
sudo mysql -e "SHOW GRANTS FOR 'app'@'localhost';"
```

Test the login over TCP with a prompt for the password:

```bash
mysql -u app -p -h 127.0.0.1 -P 3306
```

## Fix

1. Set a new password: `ALTER USER 'app'@'localhost' IDENTIFIED BY 'new-password';` and use it in the application.
2. Create the user for the host that actually connects: `CREATE USER 'app'@'192.0.2.10' IDENTIFIED BY '…';` followed by `GRANT … ON db.* TO …`.
3. For `root` on MariaDB, log in with `sudo mysql` instead of a password.
4. Update the credentials in the application's configuration and restart the application.
5. Store passwords with special characters in quotes, or choose one without `$`, `!` and `#`.

## FAQ

**I forgot the root password. What now?**

On MariaDB, `sudo mysql` usually works without a password. Otherwise restart the server briefly with `--skip-grant-tables` while it is not reachable from the network, set a new password and restart normally.

**Is the error the same as 2002?**

No. With 1045 the server answered and said no. With [2002](https://daniele-michelin.com/en/errors/mysql-error-2002-cant-connect-through-socket/) the client did not reach any server.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-1045-access-denied/](https://daniele-michelin.com/en/errors/mysql-1045-access-denied/)**
