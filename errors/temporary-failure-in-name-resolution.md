# Temporary failure in name resolution (ping, apt, curl, ssh)

> The program asked the system resolver for an IP address and got no answer at all. This is not “domain does not exist”. The server cannot reach a working DNS server, or has none configured.

Example:

```
$ ping example.com
ping: example.com: Temporary failure in name resolution
$ sudo apt update
Err:1 http://deb.debian.org/debian bookworm InRelease
  Temporary failure resolving 'deb.debian.org'
```

## Common causes

- `/etc/resolv.conf` is empty, missing or points to a dead address, for example the stub `127.0.0.53` while `systemd-resolved` is not running.
- The network itself is down: no default route, no link, or the interface did not get its DNS servers from DHCP.
- A firewall blocks outgoing DNS on port 53 (UDP and TCP).
- In a container or VM the host passes no working DNS server to it.
- A VPN or a network manager rewrote the DNS settings.

## How to check

Which DNS servers does the system use?:

```bash
resolvectl status
```

What is in resolv.conf?:

```bash
cat /etc/resolv.conf
```

Is there a default route?:

```bash
ip route show default
```

Does the network work without DNS?:

```bash
ping -c 2 1.1.1.1
```

## Fix

1. If the ping to the IP also fails, fix the network first (cable, DHCP, gateway, firewall). This is not a DNS problem then.
2. Restart the resolver: `sudo systemctl restart systemd-resolved` and check `systemctl status systemd-resolved`.
3. Restore the link to the managed file: `sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf`. This replaces the current file, so look at it first.
4. Set DNS servers permanently where the network is configured (netplan, systemd-networkd, NetworkManager or `/etc/network/interfaces`), not by editing `resolv.conf` by hand.
5. Allow outgoing UDP and TCP port 53 in the firewall.

## FAQ

**Is this the same as NXDOMAIN?**

No. NXDOMAIN is an answer: the name does not exist. Here there is no answer because the resolver cannot be reached or is not set up.

**It worked a minute ago. What changed?**

Often a DHCP renewal, a VPN or network manager change, or a stopped `systemd-resolved`. Check `resolvectl status` and the log with `journalctl -u systemd-resolved`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/temporary-failure-in-name-resolution/](https://daniele-michelin.com/en/errors/temporary-failure-in-name-resolution/)**
