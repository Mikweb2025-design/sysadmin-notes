# Factory reset a FRITZ!Box: what you lose

> A reset deletes all your own settings. It is the last resort, not the first. Make a backup beforehand if you still have access.

Example:

```
Die Box soll auf Werkseinstellungen zurückgesetzt werden, weil nichts mehr funktioniert oder das Kennwort fehlt.
```

## Common causes

- The box password is lost and there is no other way in.
- The box keeps misbehaving after changes or updates.
- The device is being handed on or sold and should be empty.
- A wrong setting cannot be found or undone any more.

## How to check

Is the box still reachable so you can back up first?:

```bash
ping 192.168.178.1
```

Does fritz.box answer?:

```bash
ping fritz.box
```

Current address of the box (Windows):

```bash
ipconfig /all
```

After the reset: test the internet connection:

```bash
ping -c 4 8.8.8.8
```

## Fix

1. If you still have access, create a settings backup in the interface under System. Menu names vary with the firmware version.
2. Write down your provider's login data: after the reset, internet access, Wi-Fi name, passwords, phone numbers and shares must be set up again (or restored from the backup).
3. A reset can be done in the interface with the factory settings function in the System area. Without access, the manual for your model describes a way via the device itself.
4. After the restart the data from the label on the underside applies again. Then set up the internet access anew or restore the backup.

## FAQ

**Does a reset delete my contract with the provider?**

No. Only the settings inside the device are erased. You do need to enter your provider login data again afterwards.

**Do I have to set a password for the backup?**

You can protect the backup with a password. Remember it well, otherwise the file cannot be restored.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/fritzbox-factory-reset/](https://daniele-michelin.com/en/errors/fritzbox-factory-reset/)**
