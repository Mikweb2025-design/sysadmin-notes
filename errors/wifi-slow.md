# Slow Wi-Fi: what can cause it and what you can check

> Your device is connected to Wi-Fi but everything feels sluggish. Often the internet line is fine and the problem is the radio path between router and device, or a crowded network.

Example:

```
Das WLAN ist verbunden, aber Seiten laden zäh und Videos ruckeln.
```

## Common causes

- Too much distance from the router, or thick walls and floors in between, weaken the signal.
- On 2.4 GHz, neighbouring networks, Bluetooth, microwaves or baby monitors interfere. This band is often crowded.
- The device sits on 2.4 GHz although 5 GHz is available. 5 GHz usually has less interference but shorter range.
- Many devices use the network at once, for example streaming, downloads and updates.
- The router is badly placed, for example in a cabinet, on the floor or behind the TV.
- The problem is not Wi-Fi at all but the internet line or DNS.

## How to check

Measure reachability and jitter to the router (Windows; on macOS/Linux use `-c 20` instead of `-n 20`):

```bash
ping -n 20 192.168.178.1
```

Check latency to the internet and compare it with the router value:

```bash
ping -n 20 8.8.8.8
```

Show signal strength and band in use on Windows:

```bash
netsh wlan show interfaces
```

List nearby Wi-Fi networks and their channels on Windows:

```bash
netsh wlan show networks mode=bssid
```

Check that name resolution answers quickly:

```bash
nslookup example.com
```

## Fix

1. First plug a cable into the router and test. If it is slow there too, Wi-Fi is not the cause: the line or the provider is.
2. If only Wi-Fi is slow, move close to the router and test again. A clear improvement points to distance or walls.
3. Place the router in the open and raised, not in a cabinet and not next to sources of interference.
4. Connect devices near the router to the 5 GHz network. On 2.4 GHz a fixed channel (1, 6 or 11) with few neighbours is often steadier than automatic.
5. Restart router and device once and install pending router firmware and Wi-Fi driver updates. Menu names differ by firmware.
6. If coverage is not enough, a repeater, a mesh system or a network cable to a fixed device can help.

## FAQ

**Is a new router the solution?**

Not automatically. Position, band and channel often help more than new hardware. Check the points above first.

**Why is Wi-Fi slower in the evening?**

More devices in your home and the neighbourhood use the radio band and the line. Either can slow things down. A test by cable shows where the problem sits.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/wifi-slow/](https://daniele-michelin.com/en/errors/wifi-slow/)**
