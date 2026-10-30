# ERROR 2006 (HY000): MySQL server has gone away

> The client sent a query on a connection that the server had already closed. It is a symptom with three common sources: the server closed an idle connection, the query was too big, or the server restarted.

Example:

```
ERROR 2006 (HY000): MySQL server has gone away
PHP Warning: mysqli::query(): MySQL server has gone away in /var/www/app/db.php on line 42
```

## Common causes

- The connection was idle longer than `wait_timeout`, the server closed it, and the app reused the dead connection, typical for long-running scripts and workers.
- A query or a row is larger than `max_allowed_packet`, for example a big import or a blob.
- The server crashed or was restarted, often killed by the OOM killer, so all connections are gone.
- A firewall, load balancer or proxy cuts idle connections earlier than the database does.
- The connection was ended with `KILL` or the server hit another limit.

## How to check

Idle timeout:

```bash
mysql -e "SHOW VARIABLES LIKE 'wait_timeout'"
```

Maximum packet size:

```bash
mysql -e "SHOW VARIABLES LIKE 'max_allowed_packet'"
```

Did the server restart? Uptime in seconds:

```bash
mysql -e "SHOW GLOBAL STATUS LIKE 'Uptime'"
```

Crash, kill or restart in the log:

```bash
journalctl -u mariadb -u mysql --since '1 day ago' --no-pager | grep -iE 'killed|out of memory|ready for connections|shutdown' | tail -n 20
```

## Fix

1. If `Uptime` is small or the log shows a kill, the server restarted. Find out why first (memory, crash). The error is only the consequence.
2. For idle connections: let the app reconnect on this error, or check the connection before use (ping). Pools should recycle connections earlier than `wait_timeout`.
3. For big queries and imports raise the limit on the server, for example `max_allowed_packet = 64M` in `[mysqld]`, and on the client too (`mysql --max-allowed-packet` or `mysqldump` setting). The value is an example, size it to your data.
4. Align the idle timeout of the firewall or proxy with `wait_timeout`, or send keepalives.
5. Do not simply set `wait_timeout` very high. It piles up idle connections and leads to Too many connections.

## FAQ

**Why does it only happen after a long pause?**

The server closed the idle connection after `wait_timeout` and the app tried to use it afterwards. The fix is on the client side: reconnect or recycle.

**It happens only on big imports.**

Then the packet is bigger than `max_allowed_packet`. Raise it on both server and client, or split the import into smaller statements.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-server-has-gone-away-2006/](https://daniele-michelin.com/en/errors/mysql-server-has-gone-away-2006/)**
