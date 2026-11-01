# Speed test shows less than your plan: why that happens

> The contract speed is what the line delivers up to the router. What reaches your device depends on Wi-Fi, cable, device and load. A single test over Wi-Fi therefore says little.

Example:

```
Der Vertrag nennt eine hohe Geschwindigkeit, der Speedtest im WLAN zeigt aber deutlich weniger.
```

## Common causes

- Wi-Fi is the bottleneck: distance, walls, interference from neighbouring networks or an old device with slow Wi-Fi.
- Many devices use the line at once (streaming, updates, cloud backup).
- An old or faulty network cable, or a port that only works at 100 Mbit/s instead of 1 Gbit/s.
- The router or the device is too weak or overloaded, or the test server is far away.
- The line itself delivers less than promised in the contract.

## How to check

Can you reach the router and how fast does it answer? (Windows: `ping -n 10 …`):

```bash
ping -c 10 192.168.178.1
```

What is the response time to the internet?:

```bash
ping -c 10 8.8.8.8
```

Which path do packets take? (Windows: `tracert example.com`):

```bash
traceroute example.com
```

Which devices are on the network? (shows address table):

```bash
arp -a
```

## Fix

1. Test by cable: connect a computer directly to the router with a LAN cable and run the speed test. Only this value shows what the line really delivers.
2. If the cable result is good but Wi-Fi is poor, Wi-Fi is the cause. Move closer to the router, improve the router's position or add an access point.
3. During the test, stop other downloads, streams and cloud backups and close unneeded programs.
4. Check the link to the router with `ping -c 10 192.168.178.1`. Large swings or losses point to Wi-Fi or the cable.
5. Swap the cable and try another port on the router.
6. If even the cable test is far below your plan over time, record several measurements at different times and bring them to your provider.

## FAQ

**Why is Wi-Fi slower than a cable?**

Wi-Fi shares the air with all devices and nearby networks and is weakened by walls and distance. A cable is steadier and usually faster.

**Is the contract speed guaranteed?**

That depends on the contract. Values are often given as a maximum or a range. Read the service description and collect measurements taken by cable.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/speed-test-too-slow/](https://daniele-michelin.com/en/errors/speed-test-too-slow/)**
