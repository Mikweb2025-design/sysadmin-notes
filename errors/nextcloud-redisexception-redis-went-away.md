# Nextcloud: RedisException: Redis went away

> Nextcloud uses Redis for file locking and caching. When the connection breaks, requests fail, uploads hang or you see 500 errors. The question is whether Redis is gone or Nextcloud cannot reach it.

Example:

```
{"reqId":"Yk3x...","level":3,"message":"RedisException: Redis went away in /var/www/nextcloud/lib/private/RedisFactory.php"}
```

## Common causes

- Redis is not running: it crashed, was stopped by an update, or was killed by the out-of-memory killer.
- The `redis` block in `config.php` points to the wrong host, port or socket path.
- Redis listens on a Unix socket, but the web server user is not in the `redis` group or `unixsocketperm` is too strict.
- Redis closes idle or overloaded connections, for example because of a low `timeout`, `maxclients` or a memory limit.

## How to check

Is Redis running?:

```bash
systemctl status redis-server --no-pager
```

Does Redis answer on the socket?:

```bash
redis-cli -s /run/redis/redis-server.sock ping
```

What does Nextcloud have configured?:

```bash
sudo -u www-data php occ config:system:get redis
```

Why did Redis stop?:

```bash
journalctl -u redis-server --since '1 hour ago' --no-pager | tail -n 20
```

## Fix

1. Start it and keep it enabled: `systemctl enable --now redis-server`.
2. Make Nextcloud use the same endpoint Redis listens on, for example `occ config:system:set redis host --value=/run/redis/redis-server.sock` and `occ config:system:set redis port --value=0 --type=integer`.
3. For a socket, add the web server user to the group: `usermod -aG redis www-data`, set `unixsocketperm 770` in `redis.conf`, then restart Redis and PHP-FPM.
4. If the kernel log shows Redis killed for memory, give the host more RAM or lower the other services. Set a `maxmemory` that fits.
5. Check `nextcloud.log` again after a few minutes to confirm the error is gone.

## FAQ

**Can I just remove Redis from config.php?**

Yes, as a test or a stopgap. Nextcloud then falls back to the database for file locking, which is slower and can cause more lock conflicts. Fix Redis rather than leaving it out.

**Is the data lost when Redis goes away?**

No. Redis holds locks and cache, not your files. After a restart the locks are rebuilt. Stuck locks from a crash can be cleaned with the file-locked entry in this lexicon.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-redisexception-redis-went-away/](https://daniele-michelin.com/en/errors/nextcloud-redisexception-redis-went-away/)**
