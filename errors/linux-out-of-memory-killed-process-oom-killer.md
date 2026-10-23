# Linux: out of memory – killed process (OOM killer)

> The system or a cgroup ran out of memory. The kernel ended the process with the highest OOM score – not necessarily the one that caused the problem.

Example:

```
Out of memory: Killed process 2314 (php-fpm8.3) total-vm:812344kB, anon-rss:402112kB, file-rss:0kB, shmem-rss:20480kB, UID:33 pgtables:1240kB oom_score_adj:0
```

## Common causes

- Too many PHP-FPM workers or database buffers for the available RAM.
- A memory leak in an application that grows over hours.
- No or too little swap, so load spikes are not absorbed.
- cgroup limit (`MemoryMax`, Docker `--memory`) reached – then “Memory cgroup out of memory”.
- One-off jobs like backups, imports or image processing need a lot of memory briefly.

## How to check

Find OOM events in the kernel log:

```bash
journalctl -k --since '24 hours ago' | grep -iE 'out of memory|oom-kill|Killed process'
```

Show free memory and swap:

```bash
free -h; swapon --show
```

Show processes using the most memory:

```bash
ps -eo pid,user,rss,comm --sort=-rss | head -n 10
```

Show the memory limits of the affected service:

```bash
systemctl show php8.3-fpm.service -p MemoryMax -p MemoryCurrent -p OOMPolicy
```

## Fix

1. Match `pm.max_children` to RAM: available memory divided by average worker size.
2. Limit database buffers, e.g. `innodb_buffer_pool_size` to fit the rest of the system.
3. Add some swap so short spikes do not cause an immediate kill.
4. Protect critical services with `OOMScoreAdjust=-500` in the unit, cap non-critical ones with `MemoryMax`.
5. Monitor memory usage over time to spot leaks early.

## FAQ

**Why was the database killed although PHP used the memory?**

The OOM killer picks by size and oom_score_adj, not by who caused it. Large services like MariaDB are therefore often hit first.

**Does disabling overcommit help?**

Rarely. With `vm.overcommit_memory=2` applications fail when allocating instead. Better set per-service limits and match usage to RAM.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/linux-out-of-memory-killed-process-oom-killer/](https://daniele-michelin.com/en/errors/linux-out-of-memory-killed-process-oom-killer/)**
