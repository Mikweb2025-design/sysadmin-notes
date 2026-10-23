# Nextcloud: "file is locked" and HTTP 423 Locked

> Nextcloud refuses access because a lock still exists for the file in transactional file locking – usually a stale lock after an aborted upload or crash.

Example:

```
OCP\Lock\LockedException: "files/9a3c1e7d/Dokumente/Bericht.docx" is locked, existing lock on file: exclusive
```

## Common causes

- File locking uses the database instead of Redis, and locks stay behind after aborts.
- An aborted upload or desktop client sync; the PHP process was killed by a timeout.
- The file is still open in Collabora or OnlyOffice.
- Background jobs do not run (no system cron), so expired locks are not cleaned up.

## How to check

Which locking backend is configured?:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get memcache.locking
```

Check that Redis is reachable:

```bash
redis-cli ping
```

Count locks in the database (with DB locking):

```bash
mysql -u nextcloud -p nextcloud -e 'SELECT COUNT(*) FROM oc_file_locks WHERE `lock` <> 0;'
```

Check the background job mode:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:app:get core backgroundjobs_mode
```

## Fix

1. Use Redis for locking: `'memcache.locking' => '\OC\Memcache\Redis'` in `config.php`.
2. Switch background jobs to system cron: `*/5 * * * * php -f /var/www/nextcloud/cron.php` in www-data's crontab.
3. Clear stale DB locks: maintenance mode on, `DELETE FROM oc_file_locks WHERE 1;`, maintenance mode off.
4. With Redis, locks expire via TTL; only flush the Redis database in an emergency, and only if Nextcloud is its sole user.
5. If needed, run `occ files:scan --path="user/files/folder"` for the affected path afterwards.

## FAQ

**Can I just disable file locking?**

Technically yes (`filelocking.enabled` set to false), but you risk corrupted files on concurrent access. Use Redis instead.

**Why does the desktop client keep hitting this?**

Usually large files plus short PHP timeouts: the upload aborts and the lock stays. Raise the timeouts and use Redis locking.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-file-is-locked-423/](https://daniele-michelin.com/en/errors/nextcloud-file-is-locked-423/)**
