# journalctl cheat sheet

Filter systemd logs precisely instead of scrolling forever – everyday journalctl commands, searchable and copyable in one click.

## Basics

- Show the last 100 lines

  ```bash
  journalctl -n 100
  ```
- Follow new entries live (like tail -f)

  ```bash
  journalctl -f
  ```
- Show newest entries first

  ```bash
  journalctl -r
  ```
- Jump to the end with explanations (after failures)

  ```bash
  journalctl -xe
  ```
- Write output to a file without pager

  ```bash
  journalctl --no-pager -n 500 > journal.txt
  ```

## Units & processes

- Show logs of a systemd unit

  ```bash
  journalctl -u nginx
  ```
- Follow a unit's logs live

  ```bash
  journalctl -u nginx -f
  ```
- Show several units interleaved

  ```bash
  journalctl -u nginx -u php8.3-fpm --since "10 min ago"
  ```
- Entries from a specific process (PID)

  ```bash
  journalctl _PID=1234
  ```
- Entries from a user (UID 33 = www-data)

  ```bash
  journalctl _UID=33
  ```
- Filter by syslog identifier

  ```bash
  journalctl -t sshd
  ```
- List all units present in the journal

  ```bash
  journalctl -F _SYSTEMD_UNIT
  ```

## Boots & kernel

- List all stored boots with index

  ```bash
  journalctl --list-boots
  ```
- Logs since the current boot

  ```bash
  journalctl -b
  ```
- Errors from the previous boot (after a crash)

  ```bash
  journalctl -b -1 -p err
  ```
- Kernel messages only (like dmesg)

  ```bash
  journalctl -k
  ```
- Look for OOM killer events in the last boot

  ```bash
  journalctl -k -b -1 | grep -i -E 'oom|killed process'
  ```

## Time, priority & search

- Entries from the last hour

  ```bash
  journalctl --since "1 hour ago"
  ```
- Narrow down an exact time window

  ```bash
  journalctl --since "2026-10-01 08:00" --until "2026-10-01 12:00"
  ```
- A unit's logs from yesterday

  ```bash
  journalctl -u nginx --since yesterday --until today
  ```
- Only errors and worse since boot

  ```bash
  journalctl -p err -b
  ```
- Priority range warning to error

  ```bash
  journalctl -p warning..err --since today
  ```
- Search by pattern (regex, case-insensitive)

  ```bash
  journalctl -u sshd -g "Failed password"
  ```

## Output formats

- Message text only, no timestamp or host

  ```bash
  journalctl -u nginx -o cat
  ```
- Entries with all fields as JSON

  ```bash
  journalctl -u nginx -o json-pretty -n 5
  ```
- ISO timestamps in UTC (for tickets)

  ```bash
  journalctl -o short-iso --utc -n 50
  ```

## Journal maintenance

- Show journal disk usage

  ```bash
  journalctl --disk-usage
  ```
- Delete archived logs older than 2 weeks (**careful**)

  ```bash
  sudo journalctl --vacuum-time=2weeks
  ```
- Shrink archived logs to 500 MB (**careful**)

  ```bash
  sudo journalctl --vacuum-size=500M
  ```
- Rotate active journal files (before vacuum)

  ```bash
  sudo journalctl --rotate
  ```
- Check journal files for corruption

  ```bash
  sudo journalctl --verify
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/journalctl-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/journalctl-cheat-sheet/)**
