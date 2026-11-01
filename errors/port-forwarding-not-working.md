# Port forwarding not working: DS-Lite, CGNAT, firewall

> A port forward sends requests from the internet to a device in your home network. It often fails not because of the setting but because your connection has no public IPv4 address at all.

Example:

```
Der Server oder die Kamera ist von außen nicht erreichbar, obwohl im Router eine Portfreigabe eingetragen ist.
```

## Common causes

- DS-Lite or CGNAT: several customers share one public IPv4 address, so classic port forwarding over IPv4 is not possible.
- The target device no longer has the same internal address because the router gave it a new one.
- The firewall on the target device or on the router blocks the port.
- Wrong port, wrong protocol (TCP or UDP) or no program is listening on that port on the target device.
- Two routers in a row: the forward is set only in the second router while the first one blocks.

## How to check

What address does the target device have? (Windows):

```bash
ipconfig /all
```

Which devices does the router know on the network?:

```bash
arp -a
```

What is the path to the internet? (Windows: `tracert example.com`) Several private addresses at the start hint at two routers or CGNAT:

```bash
traceroute example.com
```

Is a program listening on the port on the target device? (Linux):

```bash
ss -ltn
```

## Fix

1. Compare the public IPv4 address shown in the router menu with the one a what-is-my-IP website shows. If they differ, or the router's address starts with 100.64 to 100.127, you have DS-Lite or CGNAT.
2. With DS-Lite or CGNAT you have no public IPv4 address of your own, so classic port forwarding cannot work. Ask your provider for a public IPv4 address, or use IPv6 with a firewall rule, or a VPN or tunnel service.
3. Give the target device a fixed address or a fixed reservation in the router so the forward does not point at nothing.
4. Check the port and protocol in the rule. Also check the firewall on the target device and allow the port.
5. Test from outside, not from your own Wi-Fi: for example with your phone's mobile data.
6. Open only the ports you really need, and protect the service with a password and updates.

## FAQ

**How do I recognise DS-Lite or CGNAT?**

The public address in the router differs from the one shown on a what-is-my-IP site, or it starts with 100.64 to 100.127. Your provider can also confirm it.

**Is port forwarding safe?**

Every open door is a risk. Open only what is needed, keep software up to date and use strong passwords. For remote access a VPN is usually the safer choice.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/port-forwarding-not-working/](https://daniele-michelin.com/en/errors/port-forwarding-not-working/)**
