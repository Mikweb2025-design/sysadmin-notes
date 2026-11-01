# Wi-Fi repeater or mesh: what makes sense for your home?

> If Wi-Fi does not reach every room, there are several ways to fix it. A repeater is usually the simpler option, a mesh system the more complete one. Placement and connection matter most.

Example:

```
Im Keller oder Dachgeschoss gibt es kaum WLAN: Repeater kaufen oder ein Mesh-System?
```

## Common causes

- The router sits at one end of the house and the signal does not reach distant rooms.
- Thick walls, floors and reinforced concrete weaken the signal a lot.
- A repeater is too far from the router and only gets a weak signal itself, which it then passes on.
- A repeater connects to the router by radio and shares the airtime. Speed behind it often drops as a result.
- Several networks with different names make devices stick to the weak one for too long.
- A mesh setup lacks compatibility or the setup was not completed.

## How to check

Check reception at the planned spot for the repeater (Windows):

```bash
netsh wlan show interfaces
```

Test reachability of the router from there (Windows; on macOS/Linux use `-c 20` instead of `-n 20`):

```bash
ping -n 20 192.168.178.1
```

See which access point the device is connected to (BSSID, router or repeater):

```bash
netsh wlan show interfaces
```

Show all receivable networks and signal strengths:

```bash
netsh wlan show networks mode=bssid
```

Check which address is used as gateway:

```bash
ipconfig /all
```

## Fix

1. First measure reception at the spot where a repeater or node should go. It should still get good reception from the router, about halfway and not inside the dead spot itself.
2. A repeater is a cheap, simple fix for a single dead spot. A mesh system of several units suits larger homes or several floors and is managed as a whole.
3. If possible, connect the extra node to the router with a network cable. This is usually the most stable option and can improve performance a lot.
4. Avoid chaining more devices than needed. Every radio hop costs performance.
5. Set the devices up following the manufacturer's guide. Menu names and steps differ by product and firmware.
6. After setup, check in different rooms which access point the device connects to.

## FAQ

**What is better: repeater or mesh?**

In general: a repeater often suffices for one dead spot, a mesh system suits larger areas and offers unified management. Both work best with good placement.

**Does a powerline adapter or cable help?**

A network cable is usually the most reliable link. Powerline can be an alternative but depends on the house wiring.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/wifi-repeater-or-mesh/](https://daniele-michelin.com/en/errors/wifi-repeater-or-mesh/)**
