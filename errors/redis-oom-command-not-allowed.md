# Redis: OOM command not allowed when used memory > 'maxmemory'

> Memory is above `maxmemory` and the `noeviction` policy forbids evicting keys. Writes are rejected while reads still work.

Example:

```
OOM command not allowed when used memory > 'maxmemory'.
```

## Common causes

- `maxmemory-policy noeviction` is active, which is the default. Redis evicts nothing and refuses writes instead.
- Keys without a TTL fill memory, for example cache entries that never expire.
- A single large key grows without bound: a list, hash or set used as a queue.
- Fragmentation and replication buffers push `used_memory` above the limit even though the data is smaller.
- After a restart `maxmemory` is missing from the config, so Redis grows until the host runs out of memory.

## How to check

Memory, limit and policy:

```bash
redis-cli INFO memory | grep -E 'used_memory_human|maxmemory_human|maxmemory_policy'
```

Find the largest keys:

```bash
redis-cli --bigkeys
```

Count evicted keys:

```bash
redis-cli INFO stats | grep evicted_keys
```

Share of keys without TTL in a sample:

```bash
redis-cli --scan --count 1000 | head -n 1000 | while read k; do redis-cli TTL "$k"; done | grep -c '^-1$'
```

## Fix

1. Choose the policy on purpose: for a pure cache use `allkeys-lru` or `allkeys-lfu`, then Redis makes room itself.
2. Keep `noeviction` only if Redis holds data that must not be lost. Then add memory and set TTLs.
3. Set a TTL on every cache key, even the ones that look permanent.
4. Split or cap large keys; `--bigkeys` shows the candidates.
5. Keep `maxmemory` well below host RAM so snapshots and copy-on-write have room.

## FAQ

**Why does Redis not just drop old keys?**

Because `maxmemory-policy` is `noeviction`, which is the default.

**Do I lose data when I switch to allkeys-lru?**

Yes, old keys get evicted. That is fine for a cache, not for queue or session data.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/redis-oom-command-not-allowed/](https://daniele-michelin.com/en/errors/redis-oom-command-not-allowed/)**
