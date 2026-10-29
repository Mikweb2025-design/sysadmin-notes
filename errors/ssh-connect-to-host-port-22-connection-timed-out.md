# SSH: connect to host port 22: Connection timed out

> Your packets get no answer at all. Unlike “Connection refused”, nothing replies: a firewall drops the traffic, the address is wrong, or the machine is off or unreachable.

Example:

```
ssh: connect to host 203.0.113.10 port 22: Connection timed out
```

## Common causes

- A firewall or cloud security group drops port 22.
- Wrong IP address or hostname, or DNS points to an old server.
- The server is down, frozen or has lost its network.
- SSH listens on another port, or only on a different interface.
- Your IP was banned by fail2ban or a similar tool, or your network blocks outgoing port 22.

## How to check

See where the connection stops:

```bash
ssh -v user@server.example.com
```

Is the port reachable? (5 second timeout):

```bash
nc -vz -w 5 server.example.com 22
```

Does the name resolve to the address you expect?:

```bash
dig +short server.example.com
```

On the server (via console): is SSH listening?:

```bash
ss -ltnp | grep sshd
```

## Fix

1. Open the port in the firewall: `ufw allow 22/tcp`, or in the cloud provider's security group.
2. Use the right port if you changed it: `ssh -p 2222 user@host`.
3. If fail2ban banned you, unban the address: `fail2ban-client set sshd unbanip 198.51.100.7`.
4. Check the server in the provider's console: is it running, does it have network?
5. Try from another network (a mobile hotspot) to rule out a block on your side.

## FAQ

**What is the difference between timed out and refused?**

Timed out means no answer came back, typically a firewall that silently drops the traffic, or a machine that is down. Refused means the machine answered and said nothing is listening on that port.

**Could this be a brute-force protection?**

Yes. After several failed logins, tools like fail2ban block your address. Check the jail status from the server console.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/ssh-connect-to-host-port-22-connection-timed-out/](https://daniele-michelin.com/en/errors/ssh-connect-to-host-port-22-connection-timed-out/)**
