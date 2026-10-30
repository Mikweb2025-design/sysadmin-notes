# systemctl cheat sheet

Field-tested systemd commands for everyday admin work – searchable and copyable in one click. Units, timers, overrides and boot analysis.

## Status & control

- State and latest log lines of a unit

  ```bash
  systemctl status nginx
  ```
- Start a unit

  ```bash
  sudo systemctl start nginx
  ```
- Stop a unit

  ```bash
  sudo systemctl stop nginx
  ```
- Stop and start a unit again (**careful**)

  ```bash
  sudo systemctl restart nginx
  ```
- Reload the config without stopping

  ```bash
  sudo systemctl reload nginx
  ```
- Short state: active, inactive or failed

  ```bash
  systemctl is-active nginx
  ```
- Check whether the unit starts at boot

  ```bash
  systemctl is-enabled nginx
  ```

## Autostart & masking

- Enable autostart at boot

  ```bash
  sudo systemctl enable nginx
  ```
- Enable autostart and start now

  ```bash
  sudo systemctl enable --now nginx
  ```
- Disable autostart

  ```bash
  sudo systemctl disable nginx
  ```
- Fully block a unit (also as dependency) (**careful**)

  ```bash
  sudo systemctl mask nginx
  ```
- Remove the mask

  ```bash
  sudo systemctl unmask nginx
  ```
- Re-read unit files after changes

  ```bash
  sudo systemctl daemon-reload
  ```

## Finding & inspecting units

- List all failed units

  ```bash
  systemctl --failed
  ```
- List running services

  ```bash
  systemctl list-units --type=service --state=running
  ```
- Show the unit file including drop-ins

  ```bash
  systemctl cat nginx
  ```
- Dependency tree of a unit

  ```bash
  systemctl list-dependencies nginx
  ```

## Journal pointers

- Follow the logs of a unit live

  ```bash
  journalctl -u nginx -f
  ```
- Messages from the last hour

  ```bash
  journalctl -u nginx --since '1 hour ago'
  ```
- Errors since the last boot

  ```bash
  journalctl -b -p err
  ```
- Disk space used by the journal

  ```bash
  journalctl --disk-usage
  ```

## Timers

- All timers with next and last run

  ```bash
  systemctl list-timers --all
  ```
- Enable and start a timer

  ```bash
  sudo systemctl enable --now backup.timer
  ```
- Run the matching job once right now

  ```bash
  sudo systemctl start backup.service
  ```
- Timer line: every day at 03:00

  ```bash
  OnCalendar=*-*-* 03:00:00
  ```
- Timer line: catch up on missed runs after boot

  ```bash
  Persistent=true
  ```
- Validate a time expression and show next run

  ```bash
  systemd-analyze calendar 'Mon *-*-* 06:00'
  ```

## Overrides with drop-ins

- Create a drop-in (survives package updates)

  ```bash
  sudo systemctl edit nginx
  ```
- Discard all local changes to the unit (**careful**)

  ```bash
  sudo systemctl revert nginx
  ```
- Drop-in [Service]: restart on failure (RestartSec=5 waits 5 s)

  ```bash
  Restart=on-failure
  ```
- Drop-in [Service]: cap memory at 1 GB

  ```bash
  MemoryMax=1G
  ```
- Drop-in [Service]: clear the inherited ExecStart before setting a new one

  ```bash
  ExecStart=
  ```

## Analysis & targets

- Show total boot time

  ```bash
  systemd-analyze
  ```
- The 15 slowest units at boot

  ```bash
  systemd-analyze blame | head -n 15
  ```
- Sandboxing score of a unit

  ```bash
  systemd-analyze security nginx
  ```
- Boot without a graphical interface

  ```bash
  sudo systemctl set-default multi-user.target
  ```
- Switch to rescue mode (stops services) (**careful**)

  ```bash
  sudo systemctl isolate rescue.target
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/systemctl-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/systemctl-cheat-sheet/)**
