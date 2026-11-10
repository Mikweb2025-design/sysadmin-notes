# Docker: Bind for 0.0.0.0:80 failed – port is already allocated

> Docker cannot bind the host port because something already holds it: another container, a host service, or a stuck docker-proxy process.

Example:

```
Error response from daemon: driver failed programming external connectivity on endpoint web: Bind for 0.0.0.0:80 failed: port is already allocated
```

## Common causes

- Another container publishes the same host port, often after switching Compose projects.
- A host service uses the port, such as nginx, Apache or Pi-hole.
- An old container is still running because `docker compose down` was never executed.
- A container uses `network_mode: host` and also declares port mappings.
- A `docker-proxy` process from a removed container still holds the port.

## How to check

Who listens on the port?:

```bash
ss -lntp | grep ':80 '
```

Which container publishes it?:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}' | grep ':80'
```

All ports including stopped containers:

```bash
docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
```

Stuck proxy processes:

```bash
ps aux | grep -i '[d]ocker-proxy'
```

## Fix

1. Find and stop the container with `docker stop NAME`, then run `docker compose up -d` again.
2. Identify the host service and either stop it or move the container to another port, for example `8080:80`.
3. With several Compose projects, assign ports deliberately instead of mapping 80 and 443 twice.
4. Clean up old containers with `docker compose down --remove-orphans`.
5. `docker restart` does not help: the port stays taken while the other process runs.

## FAQ

**Can two containers both use port 80?**

Not directly. Only one holds the host port. Put a reverse proxy in front and use different internal ports.

**The port is taken but docker ps is empty.**

Then a host process holds it. `ss -lntp` shows the PID and program.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/docker-port-already-allocated/](https://daniele-michelin.com/en/errors/docker-port-already-allocated/)**
