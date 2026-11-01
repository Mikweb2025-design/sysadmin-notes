# Printer not found on your home network: what to do

> The printer is on, but your PC, Mac or phone cannot find it. Usually the devices are not on the same network, or the printer received a new IP address.

Example:

```
Der Drucker wird im WLAN angezeigt, aber der PC findet ihn nicht / Printer not found on the network / Stampante non trovata in rete
```

## Common causes

- PC and printer are on different networks, for example one on the guest network, or one behind a second router.
- The router gave the printer a new IP address via DHCP, while the printer port on your PC still points to the old one.
- The printer is joined to Wi-Fi but lost contact with the router, for example after a password change or router swap.
- Client or AP isolation (often part of a guest network) stops Wi-Fi devices from seeing each other.
- The PC firewall or the "Public" network profile blocks printer discovery.

## How to check

Show your PC's and router's addresses (Windows):

```bash
ipconfig /all
```

Check that the printer answers (use the IP from the printer display or router device list):

```bash
ping 192.168.178.20
```

List all devices seen on your network:

```bash
arp -a
```

Check that the router itself is reachable (your address may differ, e.g. 192.168.0.1):

```bash
ping 192.168.178.1
```

## Fix

1. Make sure PC and printer use the same Wi-Fi network. A guest network is not suitable for this.
2. Look up the printer's IP address (network report on the printer, or the router's device list) and ping it.
3. If it answers, add the printer again: on Windows go to Settings, Printers and scanners, Add a printer, then enter the IP address.
4. Give the printer a fixed address in the router (IP reservation, often called "always assign the same IP") so it never changes.
5. If it does not answer, power the printer off, restart the router and rejoin the printer to Wi-Fi. Menu names vary with router model and firmware.

## FAQ

**Why does my phone find the printer but my PC does not?**

They are often on different networks, or the PC uses the "Public" network profile. Set your home network to "Private" and check both are on the same network.

**Do I need a fixed IP for the printer?**

Not strictly, but it helps. If the address changes, the old printer port points nowhere. A reservation in the router prevents that.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/network-printer-not-found/](https://daniele-michelin.com/en/errors/network-printer-not-found/)**
