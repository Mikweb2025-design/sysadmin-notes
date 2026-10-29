# Docker: permission denied while trying to connect to the daemon socket

> Your user may not use `/var/run/docker.sock`. Only root and members of the `docker` group are allowed to talk to the Docker daemon.

Example:

```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.47/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

## Common causes

- Your user is not in the `docker` group.
- You were added to the group, but your current session still has the old group list.
- The permissions of `/var/run/docker.sock` were changed.
- A script or CI job runs as a different user than you tested with.
- With rootless Docker, the client points to the wrong socket.

## How to check

Which groups does your session have?:

```bash
id
```

Who owns the socket?:

```bash
ls -l /var/run/docker.sock
```

Is the daemon running at all?:

```bash
systemctl status docker
```

## Fix

1. Add your user to the group: `sudo usermod -aG docker $USER`.
2. Log out and in again, or start a shell with the new group: `newgrp docker`.
3. For a quick one-off, run the command with `sudo`.
4. For CI or services, add the service user to the group and restart the service.
5. Prefer rootless Docker or Podman if you do not want to hand out root-equivalent rights.

## FAQ

**Is adding a user to the docker group safe?**

It is equivalent to giving that user root rights on the host, because a container can mount any directory. Do it only for people you trust as admins.

**What if the message says it cannot connect to the daemon at all?**

That is a different error: the Docker service is not running. Start it with `systemctl start docker` and read the journal if it fails.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/docker-permission-denied-docker-sock/](https://daniele-michelin.com/en/errors/docker-permission-denied-docker-sock/)**
