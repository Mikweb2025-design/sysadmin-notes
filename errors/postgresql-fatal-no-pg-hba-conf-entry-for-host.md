# PostgreSQL: FATAL: no pg_hba.conf entry for host, user, database

> The server is reachable, but its access file `pg_hba.conf` has no line that fits this client: address, user, database and SSL mode. It rejects before it even asks for a password.

Example:

```
psql: error: connection to server at "db.example.com" (203.0.113.20), port 5432 failed: FATAL:  no pg_hba.conf entry for host "198.51.100.7", user "appuser", database "appdb", SSL encryption
```

## Common causes

- No `host` line exists for the client address, or the address range is too narrow or written wrongly.
- The user or database does not match the line, for example the line is for `all` users but a different database.
- The client uses SSL but the line is `hostnossl`, or the other way round with `hostssl`.
- The file was changed but PostgreSQL was not reloaded.
- The client comes from another address than you expect, for example a Docker network, a NAT or a proxy.

## How to check

Which rules does the server really use, in order? Also shows errors in the file:

```bash
sudo -u postgres psql -c 'SELECT line_number, type, database, user_name, address, auth_method, error FROM pg_hba_file_rules'
```

On which addresses does the server listen?:

```bash
sudo -u postgres psql -c 'SHOW listen_addresses'
```

Is the port open on the server?:

```bash
sudo ss -tlnp | grep 5432
```

The address the server saw: it is in the error text. Also in the log:

```bash
sudo tail -n 20 /var/log/postgresql/postgresql-*-main.log
```

## Fix

1. Take the address, user, database and SSL mode from the error text. They are exactly what the server saw.
2. Add one narrow line to `pg_hba.conf`, for example `hostssl appdb appuser 198.51.100.7/32 scram-sha-256`. Use the real client address or a small range.
3. Reload: `sudo systemctl reload postgresql`, then check the rule list again.
4. Do not use `0.0.0.0/0` with `trust`. That lets anyone on the network in without a password.
5. If the client is not reachable at all (connection refused), `listen_addresses` or the firewall is the next place to look.

## FAQ

**Do I need to restart PostgreSQL?**

No. A reload is enough for `pg_hba.conf`: `systemctl reload postgresql` or `SELECT pg_reload_conf();`.

**The client address in the error is not mine.**

Then a proxy, NAT or the Docker bridge changes the source address. Write the rule for the address the server reports, or fix the network path.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postgresql-fatal-no-pg-hba-conf-entry-for-host/](https://daniele-michelin.com/en/errors/postgresql-fatal-no-pg-hba-conf-entry-for-host/)**
