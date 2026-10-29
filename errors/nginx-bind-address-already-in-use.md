# nginx: bind() failed (98: Address already in use)

> nginx cannot open its port because another process already listens on it – often Apache, a second nginx or a Docker container.

Example:

```
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
```

## Common causes

- Apache, another nginx instance or a panel service (Plesk, cPanel) already listens on port 80 or 443.
- nginx is already running and was started a second time instead of reloaded.
- A Docker container publishes the same port, e.g. `-p 80:80`.
- A stale nginx master process survived a crash or a bad restart.
- Two services were set to the same `listen` address and port in your own configuration.

## How to check

Who listens on port 80 and 443?:

```bash
ss -ltnp | grep -E ':(80|443) '
```

Is the configuration valid? (a port conflict only shows up on start):

```bash
nginx -t
```

Look for leftover nginx processes:

```bash
ps aux | grep '[n]ginx'
```

Containers that publish ports:

```bash
docker ps --format '{{.Names}}  {{.Ports}}'
```

## Fix

1. Stop and disable the other service if it should not run: `systemctl disable --now apache2`.
2. If nginx itself is running, use `systemctl reload nginx` instead of starting it again.
3. Kill only a clearly stale master process, then start the service: `pkill nginx; systemctl start nginx`.
4. Move one of the services to another port, or bind each to its own IP address: `listen 203.0.113.10:80;`.
5. On a server with a panel, change the setting in the panel and do not start a second nginx by hand.

## FAQ

**nginx -t says OK but nginx still does not start. Why?**

`nginx -t` checks the syntax and files, but it does not occupy the ports. A port conflict only appears when nginx really starts, so always read the output of `ss -ltnp` too.

**Can two services share port 80?**

Only if they listen on different IP addresses. For the same address and port, one of them has to move or sit behind the other as a reverse proxy.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-bind-address-already-in-use/](https://daniele-michelin.com/en/errors/nginx-bind-address-already-in-use/)**
