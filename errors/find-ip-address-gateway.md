# How to find your IP address and gateway at home

> In a home network the gateway is almost always the router. You need its address to open the router's settings page. Your own IP address shows whether you actually joined the network.

Example:

```
Das Router-Menü lässt sich nicht öffnen, weil man die Adresse des Routers nicht kennt (z. B. 192.168.178.1).
```

## Common causes

- The manufacturer default (for example 192.168.178.1 or 192.168.0.1) was changed or does not match your router.
- Your computer did not get a valid address and shows one that starts with 169.254.
- Two routers in the house (provider router plus your own) create two separate networks, and you are in the wrong one.
- A VPN changes the routes, so the gateway shown is not the one of your home network.

## How to check

Windows: show IP address and default gateway:

```bash
ipconfig /all
```

macOS: show the gateway:

```bash
route -n get default
```

Linux: show the default route and gateway:

```bash
ip route
```

Is the gateway reachable? (Windows: `ping -n 4 …`):

```bash
ping -c 4 192.168.178.1
```

## Fix

1. On Windows run `ipconfig /all` in the command prompt and read the Default Gateway line of your active adapter (Wi-Fi or Ethernet).
2. On macOS run `route -n get default` in the Terminal and read the gateway line. Alternatively: System Settings, Network, Details, TCP/IP, Router.
3. On Linux run `ip route`: the line starting with `default via` names the gateway.
4. Type that address into your browser, for example `http://192.168.178.1`, and sign in with the router's login (often printed on the device).
5. If your own address starts with 169.254, the computer got no address from the router: check cable and Wi-Fi and restart the router.
6. With two routers, connect to the right network. The gateway is the router you are currently connected to.

## FAQ

**Is the gateway always the router?**

In a normal home network, yes. In a company network or with an active VPN the gateway can be a different device.

**My address starts with 192.168. Is that my public IP?**

No. Addresses like 192.168.x.x or 10.x.x.x are private and only valid inside your home network. The public address is shown in the router menu or on a website that displays it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/find-ip-address-gateway/](https://daniele-michelin.com/en/errors/find-ip-address-gateway/)**
