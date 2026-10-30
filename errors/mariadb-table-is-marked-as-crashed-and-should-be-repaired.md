# MariaDB / MySQL: Table is marked as crashed and should be repaired (ERROR 1194 / 145)

> The table files of a MyISAM or Aria table are inconsistent, usually because the server stopped in the middle of a write. The table is blocked until it is checked and repaired. InnoDB tables recover on their own and rarely show this.

Example:

```
ERROR 1194 (HY000): Table 'wp_options' is marked as crashed and should be repaired
ERROR 145 (HY000): Table './shop/orders' is marked as crashed and should be repaired
```

## Common causes

- The server was killed or crashed during a write: OOM killer, power loss, forced stop.
- The disk ran full while the table was written.
- The table uses MyISAM or Aria, which have no crash recovery like InnoDB.
- Table files were copied while the server was running.
- Failing disk or file system errors.

## How to check

Which engine does the table use? (replace db and table):

```bash
mysql -e "SELECT table_name, engine FROM information_schema.tables WHERE table_schema = 'shop' AND table_name = 'orders'"
```

Check the table (read lock while it runs):

```bash
mysqlcheck --check shop orders
```

Is there free space on the data disk?:

```bash
df -h /var/lib/mysql
```

Why did the server stop? Crash or kill in the log:

```bash
journalctl -u mariadb -u mysql --since '1 day ago' --no-pager | grep -iE 'crashed|killed|out of memory|aborted' | tail -n 20
```

## Fix

1. Make a copy first. If the table can still be read, `mysqldump shop orders > orders.sql`. Otherwise copy the table files while the server is stopped.
2. Repair: `mysqlcheck --repair shop orders` or in SQL `REPAIR TABLE shop.orders;`. A repair can drop rows that are damaged, so compare the row count with your backup.
3. If it does not work, stop the server and run `myisamchk` (MyISAM) or `aria_chk` (Aria) on the files. Make a copy of the files before that.
4. Restore from a backup if the repair loses too much.
5. Fix the cause (memory, disk, power) and consider converting to InnoDB: `ALTER TABLE shop.orders ENGINE=InnoDB;`. Test it first, as it changes locking and table size.

## FAQ

**Why do only some tables crash?**

Only MyISAM and Aria tables are open for writing in a way that can break at a crash. InnoDB uses a log to recover.

**Will REPAIR TABLE lose data?**

It tries to keep everything, but it can discard unreadable rows. That is why a copy comes first.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mariadb-table-is-marked-as-crashed-and-should-be-repaired/](https://daniele-michelin.com/en/errors/mariadb-table-is-marked-as-crashed-and-should-be-repaired/)**
