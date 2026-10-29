# systemd: Start request repeated too quickly (start-limit-hit)

> The service crashed again and again in a short time, so systemd stopped restarting it to protect the system. The limit is only the symptom – the service keeps failing for another reason.

Example:

```
myapp.service: Start request repeated too quickly.
myapp.service: Failed with result 'start-limit-hit'.
Failed to start myapp.service - My application.
```

## Common causes

- The program exits right after start: wrong config, missing file, permission problem or a port that is already in use.
- `Restart=always` with a very short `RestartSec` restarts a broken service too often.
- A dependency (database, mount, network) is not ready yet.
- More than 5 starts within 10 seconds, the defaults of `StartLimitBurst` and `StartLimitIntervalSec`.
- A bad update changed a binary or a configuration.

## How to check

State and last lines of the unit:

```bash
systemctl status myapp --no-pager
```

Why did it crash? Read the journal of the unit:

```bash
journalctl -u myapp -n 50 --no-pager
```

The limits that apply to this unit:

```bash
systemctl show myapp -p Restart -p RestartUSec -p StartLimitBurst -p StartLimitIntervalUSec
```

## Fix

1. Find and fix the real error in the journal first (config, permission, port, dependency).
2. Clear the failed state: `systemctl reset-failed myapp`, then `systemctl start myapp`.
3. Give the service breathing room: `RestartSec=5` in `[Service]` and, if needed, `StartLimitIntervalSec=60` and `StartLimitBurst=5` in `[Unit]`.
4. Add `After=` and `Requires=` for dependencies that must be ready first.
5. Run `systemctl daemon-reload` after editing the unit file.

## FAQ

**Can I just raise the start limit?**

You can, but it only hides a crash loop. First find out why the process exits. Raise the limit only for a service that is meant to restart often.

**How do I see all failed units?**

Run `systemctl --failed`.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/systemd-start-request-repeated-too-quickly/](https://daniele-michelin.com/en/errors/systemd-start-request-repeated-too-quickly/)**
