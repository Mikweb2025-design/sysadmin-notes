# How to restart your router properly, and when it helps

> A restart clears many small faults: stuck connections, a full connection table or a lost dial-in. The order matters, otherwise devices do not get a clean address.

Example:

```
Das Internet ist weg oder sehr langsam, Webseiten laden nicht, aber das WLAN-Symbol zeigt Verbindung.
```

## Common causes

- The router has been running for weeks and ended up in an error state (memory, connection table, dial-in).
- The link to your provider dropped and the router did not reconnect cleanly.
- After a power cut, devices started in the wrong order and did not get a valid address.
- The cause is not the router at all but a cable, a single device or the provider, so a restart does not help.

## How to check

Can you reach the router itself? (the address is often 192.168.178.1 or 192.168.0.1; see the gateway page):

```bash
ping 192.168.178.1
```

Can you reach the internet by plain address? (Windows uses `ping -n 4 8.8.8.8`):

```bash
ping -c 4 8.8.8.8
```

Does name resolution work?:

```bash
nslookup example.com
```

Did your computer get a valid address after the restart? (Windows):

```bash
ipconfig /all
```

## Fix

1. Switch off or unplug heavy devices for a moment, such as a streaming box or a game console.
2. Unplug the router from power and wait about 30 seconds so it fully shuts down and its connection tables are cleared.
3. Plug it back in and wait until the important lights (power, DSL or internet, Wi-Fi) stay on. This can take a few minutes.
4. Then bring the other devices back one by one: access points and repeaters first, then computers and phones.
5. Test with `ping 192.168.178.1` and `ping -c 4 8.8.8.8`. If the first works and the second does not, the problem is beyond the router (cable or provider).
6. If the restart only helps for a short time, note when the fault returns and check the router's event log in its menu. Do not use a factory reset unless you have saved your settings.

## FAQ

**Is restarting often bad for the router?**

An occasional restart does no harm. If you have to restart it every day, that points to a problem such as heat, a faulty power supply or old firmware, and should be investigated.

**What is the difference between a restart and a factory reset?**

A restart keeps all settings. A factory reset erases them, including your Wi-Fi password and the provider login, and is the last resort.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/restart-router-correctly/](https://daniele-michelin.com/en/errors/restart-router-correctly/)**
