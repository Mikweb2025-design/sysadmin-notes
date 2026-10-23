# Nextcloud: fixing "maintenance mode is enabled"

> Nextcloud is in maintenance mode and blocks the web UI, clients and most occ commands – either set on purpose or left behind by an update that did not finish cleanly.

Example:

```
Nextcloud is in maintenance mode, no apps are loaded.
```

## Common causes

- The web updater or a package update was aborted and `occ upgrade` never completed.
- A backup script enabled maintenance mode and failed before turning it off.
- `'maintenance' => true` is still set in `config/config.php`.
- Wrong file ownership after the update, so occ cannot write `config.php`.

## How to check

Show status and versions:

```bash
sudo -u www-data php /var/www/nextcloud/occ status
```

Check maintenance in the config:

```bash
grep -n maintenance /var/www/nextcloud/config/config.php
```

Latest errors in the Nextcloud log:

```bash
tail -n 20 /var/www/nextcloud/data/nextcloud.log
```

Is the web updater stuck?:

```bash
ls -la /var/www/nextcloud/data/updater-*/
```

## Fix

1. Finish the pending upgrade: `sudo -u www-data php occ upgrade`.
2. Then turn it off: `sudo -u www-data php occ maintenance:mode --off`.
3. Stuck web updater: remove the `.step` file in `data/updater-*/` and rerun the update with `php updater/updater.phar`.
4. Fix ownership: `chown -R www-data:www-data /var/www/nextcloud`.
5. Use `trap` in backup scripts so maintenance mode is switched off even on failure.

## FAQ

**Can I just set maintenance to false in config.php?**

Yes, but only if no upgrade is pending. Otherwise Nextcloud shows the update page or errors afterwards. Check `occ status` first.

**How is this different from maintenance:singleuser?**

Single-user mode still lets admins in through the browser; regular maintenance mode locks out everyone.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-maintenance-mode-enabled/](https://daniele-michelin.com/en/errors/nextcloud-maintenance-mode-enabled/)**
