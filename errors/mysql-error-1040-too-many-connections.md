# MySQL / MariaDB: ERROR 1040 (08004): Too many connections

> The server has reached `max_connections` and rejects every new client. Raising the limit alone rarely solves it. The question is why connections stay open: leaking apps, slow queries or a traffic peak.

Example:

```
ERROR 1040 (08004): Too many connections
SQLSTATE[HY000] [1040] Too many connections
```

## Common causes

- An application opens connections and does not close or reuse them, for example without a pool or with too many workers.
- Slow queries or locks keep connections busy, so new requests pile up behind them.
- A traffic peak or a bot flood makes a web app open many connections at once.
- Idle connections stay open too long because `wait_timeout` is high.
- `max_connections` is simply too low for the real load.

## How to check

Current limit:

```bash
mysql -e "SHOW VARIABLES LIKE 'max_connections'"
```

How high did it get since the start?:

```bash
mysql -e "SHOW GLOBAL STATUS LIKE 'Max_used_connections'"
```

What are the connections doing?:

```bash
mysql -e 'SHOW FULL PROCESSLIST'
```

Which user and host holds most of them?:

```bash
mysql -e 'SELECT user, host, COUNT(*) AS n FROM information_schema.processlist GROUP BY user, host ORDER BY n DESC'
```

## Fix

1. Log in as an admin account. The server keeps one extra connection for users with the needed privilege, so the root account can usually still get in.
2. Find the cause in the process list: many `Sleep` entries mean leaking or idle clients, many long-running queries mean slow queries or locks.
3. End stuck sessions if needed with `KILL id;`. This aborts what they are doing.
4. Fix the application: use a connection pool, close connections, limit the number of workers.
5. Only then raise the limit, for example `SET GLOBAL max_connections = 300;` and keep it in the `[mysqld]` section of the config. Every connection uses memory, so do not set it higher than the RAM allows. Consider a lower `wait_timeout` too.

## FAQ

**Is it safe to just raise max_connections?**

It hides the cause and can lead to memory pressure and the OOM killer. Raise it moderately and watch memory, after you have fixed leaks and slow queries.

**I cannot log in at all, not even as root.**

Connect through the local socket as root, which uses the extra slot. If that also fails, restart the service and fix the application before it reconnects.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-error-1040-too-many-connections/](https://daniele-michelin.com/en/errors/mysql-error-1040-too-many-connections/)**
