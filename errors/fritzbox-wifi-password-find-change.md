# FRITZ!Box: find or change the Wi-Fi password

> The Wi-Fi password (network key) is printed on the box label from the factory. You can also view and change it in the interface if you have access.

Example:

```
Ein neues Gerät soll ins WLAN, aber das WLAN-Kennwort ist unbekannt oder wird abgelehnt.
```

## Common causes

- The password was changed and not written down.
- It is mixed up with the password of the box settings.
- The device connects to the wrong network (2.4 GHz and 5 GHz often have two names).
- Typing errors: case matters, and characters such as 0 and O look alike.

## How to check

Show the saved Wi-Fi password on a Windows PC (insert the Wi-Fi name):

```bash
netsh wlan show profile name="MyWiFi" key=clear
```

Which Wi-Fi is this computer connected to?:

```bash
netsh wlan show interfaces
```

Is the box reachable so you can change the password?:

```bash
ping fritz.box
```

Read the saved password on macOS (insert the Wi-Fi name, asks for login):

```bash
security find-generic-password -wa "MyWiFi"
```

## Fix

1. Check the label on the box: it shows the Wi-Fi key if it was not changed.
2. A device that is already connected often still knows the password. On Windows the command `netsh wlan show profile` shows the key in the field Key Content.
3. To change it, sign in at fritz.box and open the Wi-Fi section, security or radio network page. Menu items vary with the firmware version.
4. Choose a long password with letters, digits and symbols, save, then reconnect all devices.
5. Check whether the 2.4 GHz and 5 GHz networks have different names and pick the right one.

## FAQ

**Do I have to reconnect all devices after changing it?**

Yes. Every device needs the new password, including printers, TVs and smart-home devices.

**Is WPS a good alternative?**

Yes, for a quick connection. The button on the box and the function on the device must be started within a short time of each other; the details depend on the model.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/fritzbox-wifi-password-find-change/](https://daniele-michelin.com/en/errors/fritzbox-wifi-password-find-change/)**
