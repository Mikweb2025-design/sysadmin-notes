# PostgreSQL: FATAL – sorry, too many clients already

> PostgreSQL has used every connection slot and refuses new clients until one is freed. The cause is almost always a pool or a leak, not the limit itself.

Example:

```
FATAL: sorry, too many clients already
connection to server at "db" (10.0.0.5), port 5432 failed
```

## Common causes

- The application opens connections and never closes them, for example after a deploy or in a worker without `finally`.
- The sum of all pool sizes exceeds `max_connections`, especially with several instances or containers.
- A PgBouncer in `session` mode passes every client connection through and defeats pooling.
- Transactions stay `idle in transaction` and hold slots without doing work.
- Backup, monitoring and migration jobs add to the application pools.

## How to check

Used and allowed connections:

```bash
psql -c "SELECT count(*) AS used, current_setting('max_connections')::int AS max FROM pg_stat_activity;"
```

Who holds the connections?:

```bash
psql -c "SELECT usename, state, count(*) FROM pg_stat_activity GROUP BY 1,2 ORDER BY 3 DESC;"
```

Long idle transactions:

```bash
psql -c "SELECT pid, now()-state_change AS idle_for, query FROM pg_stat_activity WHERE state LIKE 'idle in transaction%' ORDER BY 2 DESC LIMIT 10;"
```

Reserved slots:

```bash
psql -c "SHOW superuser_reserved_connections;"
```

## Fix

1. Measure first, then raise: a bigger `max_connections` only hides a leak and costs memory per backend.
2. Close the leak: use context managers or `finally`, and keep each pool smaller than `max_connections / instances`.
3. For many short connections put PgBouncer in `transaction` mode in front.
4. Set `idle_in_transaction_session_timeout` so stuck transactions are terminated.
5. Raise the limit only with memory headroom for autovacuum and backups, and remember it needs a restart.

## FAQ

**Is raising max_connections enough?**

Briefly, yes. With a leak the error returns within hours. Measure who holds the slots first.

**Why are fewer connections free than max_connections suggests?**

Reserved slots for superusers and background processes count against the limit, so a normal client hits it earlier.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/postgres-too-many-clients/](https://daniele-michelin.com/en/errors/postgres-too-many-clients/)**
