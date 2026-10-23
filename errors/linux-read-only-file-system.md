# Linux: read-only file system (EROFS)

> A write was refused because the filesystem is read-only. Often the kernel switched it itself after an error to prevent further damage.

Example:

```
touch: cannot touch '/var/www/test': Read-only file system
```

## Common causes

- ext4 with `errors=remount-ro` detected an error and switched to read-only.
- Failing disk or SSD, I/O errors in the VM's storage backend.
- Mounted `ro` in `/etc/fstab`, or a deliberately read-only root (containers, appliances).
- systemd hardening: `ProtectSystem=strict` makes everything except `ReadWritePaths` read-only for the service.
- Thin-provisioned storage full on the hypervisor.

## How to check

Show mount options of the affected path:

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS -T /var/www
```

Search kernel messages for filesystem and I/O errors:

```bash
dmesg -T | grep -iE 'EXT4-fs error|I/O error|remount|read-only' | tail -n 20
```

Check the disk's SMART status:

```bash
smartctl -H -A /dev/sda
```

Show the service's systemd hardening:

```bash
systemctl show nginx.service -p ProtectSystem -p ReadWritePaths -p ProtectHome
```

## Fix

1. Back up data first – while it can still be read.
2. For filesystem errors run `fsck -f` on the unmounted device from a rescue system.
3. Only if the cause is clear and harmless: `mount -o remount,rw /`.
4. For systemd services allow the needed paths via override: `ReadWritePaths=/var/www/uploads`.
5. With I/O or SMART errors replace the disk or contact the hoster.

## FAQ

**Can I just run remount,rw?**

Technically yes, but if the kernel switched because of errors you are writing to a damaged filesystem. Check first, then remount.

**Why can root write but the service cannot?**

Then systemd hardening is usually active. ProtectSystem and ReadOnlyPaths only apply in the service's namespace, not in your shell.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/linux-read-only-file-system/](https://daniele-michelin.com/en/errors/linux-read-only-file-system/)**
