# Wi-Fi keeps disconnecting: causes and checks

> The connection is up, drops for a moment and comes back. That usually points to a weak or disturbed radio signal, a device problem or an overloaded router.

Example:

```
Das WLAN trennt sich immer wieder, besonders in weiter entfernten Räumen oder zu bestimmten Zeiten.
```

## Common causes

- The signal is weak at the edge of range and fluctuates, so the device loses the connection.
- Interference on the channel in use, especially on 2.4 GHz, makes the connection drop now and then.
- The power-saving mode of the Wi-Fi adapter briefly switches the connection off on a laptop.
- The router has been running a long time without restart, is overheated or has outdated firmware.
- The device switches badly between 2.4 and 5 GHz or between router and repeater.
- Outdated or faulty Wi-Fi driver in the device.

## How to check

Check when the router stops answering (Windows; on macOS/Linux use `-c 60` instead of `-n 60`):

```bash
ping -n 60 192.168.178.1
```

Show signal strength, band and channel of the device:

```bash
netsh wlan show interfaces
```

Create a report of Wi-Fi events (run as administrator):

```bash
netsh wlan show wlanreport
```

Show current IP settings and gateway:

```bash
ipconfig /all
```

Check whether only Wi-Fi is affected: internet reachability:

```bash
ping -n 20 8.8.8.8
```

## Fix

1. Note when and where it happens: only in one room, only one device, or all devices. This narrows the cause a lot.
2. If all devices are affected, restart the router and check for firmware updates. Menu names differ by router.
3. If only one device is affected, update the Wi-Fi driver and switch off power saving for the adapter in its device properties.
4. Move closer to the router or improve its position. Walls and distance are the most common cause.
5. With a lot of interference on 2.4 GHz, pick a fixed channel (1, 6 or 11) or use the 5 GHz band near the router.
6. If drops are linked to a repeater, check where it sits. It should still get good reception from the router.

## FAQ

**Is a drop every few minutes normal?**

No. Occasional glitches can happen, but regular drops almost always have a cause such as interference, signal or driver.

**Should I restart the router regularly?**

That is only a workaround. If a restart always helps, check the firmware and whether the router has good ventilation.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/wifi-keeps-disconnecting/](https://daniele-michelin.com/en/errors/wifi-keeps-disconnecting/)**
