# ERROR 2003 (HY000): Can't connect to MySQL server (10061)

> The client reached the host but the port refused the connection: the service is not running, listens on `127.0.0.1` only, or a firewall drops the packet.

Example:

```
ERROR 2003 (HY000): Can't connect to MySQL server on 'db.example.com' (10061)
```

## Common causes

- The database service is not running or crashed, for example after an update or an OOM kill.
- `bind-address = 127.0.0.1` in the config, so remote connections are refused.
- Wrong port or hostname, or the client connects through a socket instead of TCP.
- `skip-networking` is set, so there is no TCP listener at all.
- A firewall or security group blocks port 3306, often just for the new client IP.

## How to check

Is the service running?:

```bash
systemctl status mariadb --no-pager | head -n 5
```

Which address and port does it listen on?:

```bash
ss -lntp | grep 3306
```

Test the port from the client:

```bash
nc -vz db.example.com 3306
```

Check bind-address and skip-networking:

```bash
grep -rE '^(bind-address|skip-networking|port)' /etc/mysql/
```

## Fix

1. Start the service and read the journal for the reason it stopped. A restart without a cause only buys time.
2. For remote clients set `bind-address = 0.0.0.0` or the specific interface IP and restart. That opens the port on purpose.
3. Open the firewall for the client IP only, not for whole networks.
4. On Windows, 10061 is usually the Windows firewall or a stopped service; 10060 is a timeout, so a wrong address or a dropping filter.

## FAQ

**What is the difference between 2002 and 2003?**

2002 is about the local socket, 2003 is the TCP connection to a host.

**10061 versus 10060?**

10061 means connection refused, so no listener. 10060 is a timeout, so a firewall or a wrong address.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-2003-cant-connect-10061/](https://daniele-michelin.com/en/errors/mysql-2003-cant-connect-10061/)**
