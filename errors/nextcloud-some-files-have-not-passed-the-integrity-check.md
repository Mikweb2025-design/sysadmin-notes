# Nextcloud: Some files have not passed the integrity check

> Nextcloud compares its code files with a signed list of hashes. Anything that was changed, deleted or added inside the program folder is reported. It is a warning about your files, not a data loss.

Example:

```
There were problems with the code integrity check. More information…
Results:
- core
  - INVALID_HASH
    - core/js/mimetypelist.js
  - EXTRA_FILE
    - core/old-backup.php
```

## Common causes

- Files were left over or mixed after a manual update, for example an old folder copied over a new one.
- A core or app file was edited by hand, or a backup, patch or `.orig` file was put into the program folder.
- An upload or unzip was incomplete, so files are missing (`MISSING_FILE`) or damaged (`INVALID_HASH`).
- An app that did not come from the App Store, or a custom theme placed in the wrong folder, is reported as extra.

## How to check

Check the core files:

```bash
sudo -u www-data php occ integrity:check-core
```

Check one app:

```bash
sudo -u www-data php occ integrity:check-app calendar
```

Which version is installed?:

```bash
sudo -u www-data php occ status
```

Files changed in the last days:

```bash
find /var/www/nextcloud -maxdepth 2 -type f -mtime -3 -not -path '*/data/*' | head -n 20
```

## Fix

1. Read the result: `INVALID_HASH` means the file was changed, `MISSING_FILE` means it is gone, `EXTRA_FILE` means it does not belong there.
2. Download the archive of exactly your installed version from the official source, verify its checksum, and copy the reported files back from it.
3. Move `EXTRA_FILE` entries out of the program folder, for example to a backup folder outside the web root.
4. Put custom themes and apps in `themes/` and `custom_apps/`, not into core folders. Use `updater/updater.phar` or the web updater for future updates.
5. Run `occ integrity:check-core` again, then press the rescan button in the admin overview.

## FAQ

**Can I ignore this warning?**

It can point to a broken or tampered installation, so do not leave it. If the list holds only a file you changed on purpose, document it. Otherwise restore the original so updates and support stay predictable.

**Does the check touch my data directory?**

No. It only looks at the program files, the shipped apps and the signed apps. Your data folder is not compared.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-some-files-have-not-passed-the-integrity-check/](https://daniele-michelin.com/en/errors/nextcloud-some-files-have-not-passed-the-integrity-check/)**
