# Two routers in a row: how to spot double NAT

> When one router sits behind another, each translates the addresses once. This is called double NAT. Port forwarding, online games and remote access then often fail.

Example:

```
Hinter dem zweiten Router klappen Portfreigaben, Spiele oder Fernzugriff nicht / Port forwarding fails behind a second router / Con due router il port forwarding non funziona
```

## Common causes

- The provider's router and your own second router are chained, and both act as routers with their own network.
- The second router gets a private address on its WAN port (for example 192.168.x.x) rather than a public one.
- A port forward was set up only in the second router, not in the first.
- Both routers use the same address range, for example both 192.168.178.x, which confuses connections.

## How to check

Show the default gateway and your PC's address (Windows):

```bash
ipconfig /all
```

Trace the path to the internet. Two private hops at the start suggest double NAT:

```bash
tracert example.com
```

Test the first router (address may differ):

```bash
ping 192.168.178.1
```

Test the second router (address from its manual):

```bash
ping 192.168.0.1
```

## Fix

1. Check in the second router what address its WAN port has. If it starts with 192.168, 10. or 172.16 to 172.31, you have double NAT.
2. Best fix: switch the second router to access point mode (or bridge mode) so the first one does the routing.
3. Connect a LAN port of the first router to a LAN port of the second, not its WAN port. The second router's DHCP server is turned off.
4. Alternative: set up the port forward in both routers, first to the second router's address, then on to the target device. This is more awkward.
5. If both stay in router mode, use different address ranges. Menu names vary with firmware.

## FAQ

**How do I recognise double NAT?**

The WAN address of your second router is a private address. A traceroute also shows two private hops in a row at the start.

**Does double NAT hurt normal browsing?**

Usually not. Mostly affected are port forwarding, online games, VPN and remote access.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/two-routers-double-nat/](https://daniele-michelin.com/en/errors/two-routers-double-nat/)**
