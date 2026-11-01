# fritz.box won't open: what to do

> The name fritz.box only works inside your home network and needs a working link to the box. The IP address 192.168.178.1 often works when the name does not.

Example:

```
Im Browser erscheint bei fritz.box „Die Seite ist nicht erreichbar“ oder „Server nicht gefunden“.
```

## Common causes

- The device is not connected to the box at all (other Wi-Fi, mobile data, guest network).
- A VPN or a custom DNS server prevents the name fritz.box from resolving.
- The box uses an address other than 192.168.178.1, for example after a change or with another router.
- Browser cache, a proxy or extensions block the page.
- The box is stuck and needs a restart.

## How to check

Does the box answer?:

```bash
ping 192.168.178.1
```

Which address does your router have (Windows, Default Gateway)?:

```bash
ipconfig /all
```

Is the name fritz.box resolved?:

```bash
nslookup fritz.box
```

Router address on macOS/Linux:

```bash
ip route show default
```

## Fix

1. Make sure your device is on the right Wi-Fi or plugged in by cable to the box, and that no VPN is running.
2. Open `http://192.168.178.1` directly instead of fritz.box. If that works, the problem is name resolution.
3. If `ipconfig /all` shows a different default gateway, type that address in the browser.
4. Try a private browser window or another browser.
5. Restart the box by unplugging the power briefly, plug it in again and wait a few minutes.

## FAQ

**Why does 192.168.178.1 work but fritz.box does not?**

Your device reaches the box but cannot resolve the name. Usually a foreign DNS server or a VPN is the reason.

**Can I open fritz.box when away from home?**

No, the name only works inside the home network. From outside you need a remote access that was set up and secured beforehand.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/fritzbox-not-reachable/](https://daniele-michelin.com/en/errors/fritzbox-not-reachable/)**
