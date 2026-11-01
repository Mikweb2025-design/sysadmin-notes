# Set up a guest network on your router: how and why

> A guest network gives visitors internet but keeps them apart from your home network. That separation is intended, but it also means printers, TVs and NAS are not reachable from it.

Example:

```
Besucher sollen ins WLAN, aber nicht an Drucker und NAS / Visitors need Wi-Fi without access to printer and NAS / Gli ospiti devono usare il Wi-Fi senza vedere stampante e NAS
```

## Common causes

- The guest network is isolated on purpose: devices on it see neither each other nor devices on the home network.
- Your own devices were put on the guest network by mistake and no longer find the printer or TV.
- The guest network is not enabled or has no password of its own, so visitors end up on your main network.
- Smart home devices are on the guest network but need access to an app or hub on the home network.

## How to check

Show your device's and the router's address. On the guest network it usually lies in a different range:

```bash
ipconfig /all
```

Check that the router is reachable (address may differ):

```bash
ping 192.168.178.1
```

Check whether a home device is reachable from the guest network. It should not answer:

```bash
ping 192.168.178.20
```

Show current Wi-Fi and signal on a PC (Windows):

```bash
netsh wlan show interfaces
```

## Fix

1. Open your router's settings and look for "Guest access", "Guest network" or "Guest Wi-Fi". The name depends on the router.
2. Enable it, choose its own name and its own strong password using WPA2 or WPA3.
3. Keep the option on that separates guest devices from each other and blocks access to the home network.
4. Your own devices such as printer, TV and NAS belong on the normal home network, not the guest network.
5. Switch the guest network off when not needed, or change its password from time to time.

## FAQ

**Why should I use a guest network?**

Strangers' phones and laptops then cannot see your files or devices. It is a simple protection if a guest device is infected.

**Can I print from the guest network?**

Usually not, because the guest network is isolated. That is intentional. Guests can print via email or USB instead.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/set-up-guest-wifi-network/](https://daniele-michelin.com/en/errors/set-up-guest-wifi-network/)**
