# MySQL / MariaDB: ERROR 2002 (HY000) can't connect through socket

> The client looks for the server's Unix socket file and does not find a running server there – usually the database is stopped, or client and server use different socket paths.

Example:

```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
```

## Common causes

- The database service is not running: stopped, crashed, or killed by the OOM killer.
- The server did not start because the disk is full or the data directory is damaged or has wrong permissions.
- Client and server use different socket paths, e.g. `/run/mysqld/mysqld.sock` and `/var/lib/mysql/mysql.sock`.
- The application connects to `localhost`, which means the socket, while the server only listens on TCP.
- A second instance or a container uses another socket.

## How to check

Is the service running?:

```bash
systemctl status mariadb
```

Why did it stop? Read the latest log lines:

```bash
journalctl -u mariadb --since '1 hour ago' --no-pager | tail -n 30
```

Does the socket exist?:

```bash
ls -l /run/mysqld/
```

Is there free space for the data directory?:

```bash
df -h /var/lib/mysql
```

## Fix

1. Start the service and read why it stopped: `systemctl start mariadb`. On MySQL the unit is called `mysql`.
2. Free disk space or repair permissions on `/var/lib/mysql` if that was the reason.
3. If the OOM killer stopped it, reduce memory use (`innodb_buffer_pool_size`) or add RAM.
4. Use the same socket path in the `[client]` and `[mysqld]` sections of the configuration.
5. In the application, use `127.0.0.1` and port 3306 instead of `localhost` to connect over TCP.

## FAQ

**What is the difference between localhost and 127.0.0.1 here?**

With `localhost` the MySQL client uses the Unix socket file. With `127.0.0.1` it uses TCP. If the socket is missing but TCP works, that explains this error.

**Which error do I get when the password is wrong?**

That is [ERROR 1045](https://daniele-michelin.com/en/errors/mysql-1045-access-denied/). Error 2002 comes before: the client does not even reach the server.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-error-2002-cant-connect-through-socket/](https://daniele-michelin.com/en/errors/mysql-error-2002-cant-connect-through-socket/)**
