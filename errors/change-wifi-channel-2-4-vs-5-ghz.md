# Change the Wi-Fi channel: 2.4 GHz or 5 GHz, which fits?

> Many Wi-Fi problems come from networks interfering with each other. The right band and a free channel often make the connection steadier.

Example:

```
Viele Nachbarnetze auf demselben Kanal, das WLAN ist langsam oder instabil, 2,4 oder 5 GHz unklar.
```

## Common causes

- On 2.4 GHz the channels overlap. Only 1, 6 and 11 do not interfere with each other.
- Many neighbouring networks and devices use the same channel, especially in apartment buildings.
- The 5 GHz band usually has less interference but shorter range through walls.
- The router's automatic channel choice picks badly or switches during use.
- The device supports only 2.4 GHz and cannot see the 5 GHz network at all.
- Router and repeater use the same channel and interfere with each other.

## How to check

Show nearby networks, channels and bands on Windows:

```bash
netsh wlan show networks mode=bssid
```

Show your own band, channel and signal strength:

```bash
netsh wlan show interfaces
```

Check reachability of the router (Windows; on macOS/Linux use `-c 20` instead of `-n 20`):

```bash
ping -n 20 192.168.178.1
```

Check whether the device supports both bands (driver info, radio types):

```bash
netsh wlan show drivers
```

## Fix

1. Use the command above to see which channels your neighbours use. For 2.4 GHz pick channel 1, 6 or 11 with the fewest networks.
2. Open your router's page in a browser (the address is usually on the device) and look for the section on Wi-Fi, wireless or radio channel. Names vary with the firmware.
3. Set 2.4 GHz to a fixed channel and save. Then test again.
4. Use 5 GHz for devices near the router, such as laptop and TV. Use 2.4 GHz for far-away devices and simple smart-home gadgets.
5. If needed, give the two bands different names so you can choose on purpose. Some routers also offer a shared name, depending on firmware.
6. If it does not improve, try another of the three channels and watch for a few days.

## FAQ

**Which channel is best?**

On 2.4 GHz it is usually one of 1, 6 or 11, the one with the fewest neighbours. On 5 GHz there is more choice and automatic selection often works well.

**Why can't I see the 5 GHz network?**

The device may not support it, or you are too far away. 5 GHz has less range. Also check that it is switched on in the router.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/change-wifi-channel-2-4-vs-5-ghz/](https://daniele-michelin.com/en/errors/change-wifi-channel-2-4-vs-5-ghz/)**
