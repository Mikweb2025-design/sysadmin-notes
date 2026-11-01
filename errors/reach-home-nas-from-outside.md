# Reach your NAS or home PC from outside the house

> You only get into the home network from outside if the router allows it. A VPN is the safest way. Port forwarding to the internet is possible but riskier.

Example:

```
Das NAS oder der Heim-PC ist von außerhalb nicht erreichbar / Home NAS not reachable from outside / NAS o PC di casa non raggiungibile da fuori
```

## Common causes

- The router blocks incoming connections by default until a forward or VPN is set up.
- Your public IP address changes regularly, so the address you noted stops working after a while.
- The connection sits behind double NAT or provider-side NAT, so nothing from outside reaches your router.
- The port forward points to a wrong or since-changed internal IP address of the NAS.

## How to check

Check the NAS's internal address on the home network:

```bash
ping 192.168.178.20
```

Check that a DynDNS name points to your current address (replace example.com with your name):

```bash
nslookup example.com
```

List devices on the home network to find the NAS address:

```bash
arp -a
```

Look at the path to the internet. Several private hops suggest double NAT:

```bash
tracert example.com
```

## Fix

1. Prefer a VPN: many routers offer a VPN server (for example WireGuard or IPsec). Connect to it and you are on the home network as if at home.
2. If your connection has no public address of its own (provider-side NAT), ask the provider for one or use a VPN service.
3. Changing address: set up a DynDNS service that points a fixed name at your changing address. The router or NAS reports the address itself.
4. Only if needed, set up a port forward, give the NAS a fixed IP (reservation) first, and use a strong password, two-factor login and current firmware.
5. Never expose admin pages or file shares (such as SMB) directly to the internet. Menu names vary with router.

## FAQ

**Is a port forward to my NAS dangerous?**

It can be. Anyone on the internet can reach the open port, and attacks on known flaws are common. A VPN keeps the NAS invisible and is preferable.

**What do I need DynDNS for?**

Your public address usually changes. DynDNS ties a fixed name to the current address so you do not have to look it up each time.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/reach-home-nas-from-outside/](https://daniele-michelin.com/en/errors/reach-home-nas-from-outside/)**
