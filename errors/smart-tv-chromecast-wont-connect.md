# Smart TV or Chromecast will not connect to Wi-Fi

> Streaming from phone to TV only works when both are on the same local network. Casting and AirPlay use mDNS, which does not cross network boundaries.

Example:

```
Das Handy findet den Chromecast oder Smart-TV nicht zum Streamen / Phone cannot find the TV to cast / Il telefono non trova la TV per il casting
```

## Common causes

- Phone and TV are on different networks, for example the phone is on the guest network or on a second router.
- The Chromecast or TV only supports 2.4 GHz or has trouble with the 5 GHz network and cannot find your Wi-Fi.
- The router isolates Wi-Fi devices from each other (guest network or AP isolation), so discovery finds nothing.
- After a router swap or password change the device does not know the new Wi-Fi.
- The signal is too weak because the TV is far from the router.

## How to check

Show your PC's network and router addresses (Windows):

```bash
ipconfig /all
```

Check that the TV answers (IP from the TV's network settings or the router device list):

```bash
ping 192.168.178.30
```

List devices on the same network:

```bash
arp -a
```

Check the signal of your current Wi-Fi on a PC (Windows):

```bash
netsh wlan show interfaces
```

## Fix

1. Join phone and TV to the same Wi-Fi. A guest network is not suitable for casting.
2. In the TV's network settings, check which Wi-Fi and IP it uses. If the address range differs from your phone's, they are on different networks.
3. If a second router is chained in, connect both devices to the same router.
4. If the device supports only 2.4 GHz, enable the 2.4 GHz band in the router or temporarily give each band its own name.
5. Restart TV or Chromecast and the router, and move the TV closer to the router. Menu names vary with firmware.

## FAQ

**Why does my phone only sometimes find the Chromecast?**

The phone may switch between 2.4 and 5 GHz or between router and repeater. Keep it on the same network as the Chromecast.

**Does casting work over the guest network?**

Usually not, because guest networks separate devices. Use your normal home network for this.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/smart-tv-chromecast-wont-connect/](https://daniele-michelin.com/en/errors/smart-tv-chromecast-wont-connect/)**
