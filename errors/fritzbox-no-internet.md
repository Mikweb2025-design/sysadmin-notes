# FRITZ!Box has no internet: what you can check yourself

> Wi-Fi works but nothing reaches the internet. The key question is whether one device, the whole box or the line is affected.

Example:

```
Das WLAN ist da, aber Webseiten laden nicht. An der Box leuchtet die Internet-Anzeige nicht oder blinkt.
```

## Common causes

- The line (DSL, fibre, cable) has a fault or is under maintenance.
- The box froze or did not start cleanly after a power cut.
- A cable between the box and the wall socket or fibre modem is loose or faulty.
- Only name resolution (DNS) is broken while the line itself is up.
- Only one device has a problem, for example because of wrong network settings.

## How to check

Is the box reachable on the home network?:

```bash
ping 192.168.178.1
```

Is there internet without name resolution?:

```bash
ping -c 4 8.8.8.8
```

Does name resolution work?:

```bash
nslookup example.com
```

Where does the path break off?:

```bash
tracert example.com
```

## Fix

1. Look at the box lights: a steady internet or DSL light is normal; blinking or dark points to a line problem.
2. Check whether several devices are affected. If only one, the problem is that device and not the box.
3. Restart the box: pull the power plug, wait about half a minute, plug it in and be patient for a few minutes.
4. Check that the cables to the wall socket or fibre modem sit firmly.
5. If `ping 8.8.8.8` answers but `nslookup example.com` does not, it is a DNS problem; try another DNS server in the network settings.
6. If the line light stays dark or keeps blinking, call your provider and tell them what you already checked.

## FAQ

**When should I call the provider?**

When, after a restart and cable check, the internet or line light still does not stay on, or several devices have no internet.

**Does a factory reset help?**

Rarely. A restart is usually enough. A reset erases all settings and should be the last step.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/fritzbox-no-internet/](https://daniele-michelin.com/en/errors/fritzbox-no-internet/)**
