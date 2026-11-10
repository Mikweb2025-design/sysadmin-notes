# ssh: connect to host … port 22: Connection refused

> The packet arrived and was actively rejected: no service listens on that port. That is what separates this error from a firewall timeout.

Example:

```
ssh: connect to host server.example.com port 22: Connection refused
```

## Common causes

- sshd is not running, for example after an update or a failed config test.
- The service listens on a different port, such as `Port 2222`, while the client uses 22.
- sshd listens on a single address or IPv6 only, so the IPv4 address has no listener.
- A firewall rejects with REJECT. DROP would produce a timeout instead.
- The hostname points to a different server where SSH is not running.

## How to check

Status and listener:

```bash
systemctl status ssh --no-pager | head -n 5; ss -lntp | grep ssh
```

Configured ports and addresses:

```bash
grep -iE '^\s*(Port|ListenAddress|AddressFamily)' /etc/ssh/sshd_config
```

Test the port from the client:

```bash
nc -vz server.example.com 22
```

Check bans on the server:

```bash
fail2ban-client status sshd 2>/dev/null | tail -n 5
```

## Fix

1. Start the service and read the journal: `systemctl start ssh`, then `journalctl -u ssh -n 30`.
2. If the port changed, update client, firewall and documentation – both sides must match.
3. With fail2ban, unban your own IP: `fail2ban-client set sshd unbanip IP`.
4. For IPv6-only listening set `AddressFamily any` or check the IPv4 address.
5. Run `sshd -t` before restarting so a broken config does not lock everyone out.

## FAQ

**Refused versus timeout?**

Refused means the packet arrived and was rejected, so there is no listener. Timeout means no answer at all: firewall DROP or a wrong address.

**I locked myself out – now what?**

Log in through the provider console, check the service and config, and unban your IP. Fail2ban only affects new connections.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssh-connection-refused-port-22/](https://daniele-michelin.com/en/errors/ssh-connection-refused-port-22/)**
