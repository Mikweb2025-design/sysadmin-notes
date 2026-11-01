# Wi-Fi connected but no internet: causes and checks

> The device is connected to the router, but websites and apps do not load. The fault can be in the device, the router, the provider's line or name resolution.

Example:

```
WLAN verbunden, aber keine Internetverbindung oder: Verbunden, kein Internet.
```

## Common causes

- The internet line or the router's login to the provider is down, for example due to an outage.
- The router has hung and needs a restart.
- The device has no valid IP address or a faulty one (169.254.x.x).
- DNS resolution does not work although the connection itself is fine.
- A guest network or public Wi-Fi requires a login on a web page first.
- A VPN or proxy on the device redirects traffic incorrectly.

## How to check

Is the router reachable? (Windows; on macOS/Linux use `-c 4` instead of `-n 4`):

```bash
ping -n 4 192.168.178.1
```

Is the internet reachable by IP address, without name resolution?:

```bash
ping -n 4 8.8.8.8
```

Does name resolution work?:

```bash
nslookup example.com
```

Show the device's IP address, gateway and DNS servers:

```bash
ipconfig /all
```

Follow the path to the internet step by step:

```bash
tracert example.com
```

## Fix

1. Check whether other devices on the same Wi-Fi get online. If not, the router or line is the cause; if yes, the single device is.
2. If the router does not answer: restart it and wait a few minutes until all lights are steady. Check for an outage with your provider, for example over mobile data.
3. If the router answers but 8.8.8.8 does not: the problem is the internet connection. Check the router's indicators and the provider's outage information.
4. If 8.8.8.8 answers but nslookup fails: it is a DNS problem. As a test, enter a different DNS server in the device's network settings or in the router.
5. If the device has a 169.254.x.x address: disconnect Wi-Fi and reconnect, or restart the device so it gets an address from the router.
6. Switch off VPN or proxy as a test, and on guest or hotel Wi-Fi open any http page to see the login page.

## FAQ

**What does 169.254.x.x mean?**

The device did not get an address from the router and made one up. The cause is usually the link to the router. Reconnecting or restarting the router often helps.

**Why does ping work but no website?**

Then the connection works but usually not name resolution (DNS). The nslookup test shows it. A different DNS server can fix it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/wifi-connected-no-internet/](https://daniele-michelin.com/en/errors/wifi-connected-no-internet/)**
