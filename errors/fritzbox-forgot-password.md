# FRITZ!Box password forgotten: how to get back in

> You can open the login page but no longer know the password. If it was never changed, the default password is printed on the underside of the device.

Example:

```
Die Anmeldeseite von fritz.box verlangt ein Kennwort, und das Kennwort ist nicht mehr bekannt.
```

## Common causes

- The password was changed at some point and never written down.
- The settings password is confused with the Wi-Fi password.
- The browser saved an old password and fills it in wrongly.
- Someone else (family member, technician) set up the box.

## How to check

Is the box reachable on your network?:

```bash
ping fritz.box
```

Show the box address (Windows, line Default Gateway):

```bash
ipconfig /all
```

Test the standard address directly:

```bash
ping 192.168.178.1
```

Show the box address (macOS/Linux):

```bash
ip route show default
```

## Fix

1. Look at the label on the underside of the FRITZ!Box: it lists the default password and the Wi-Fi key if you never changed them.
2. Try the login in a private browser window so saved old data does not interfere.
3. The login page usually offers a "forgot password" option; it normally asks you to press a button on the box to confirm you have physical access. Wording varies with the firmware version.
4. If nothing helps, the last resort is a factory reset. It erases all settings (see the factory reset article).

## FAQ

**Is the box password the same as the Wi-Fi password?**

Not necessarily. The Wi-Fi password is the network key for devices; the box password protects the settings. They can differ from the factory.

**Can I display the password?**

No, passwords are not shown in clear text. You can only use the default one from the label or set a new one.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/fritzbox-forgot-password/](https://daniele-michelin.com/en/errors/fritzbox-forgot-password/)**
