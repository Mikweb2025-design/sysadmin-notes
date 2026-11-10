# MariaDB/MySQL: ERROR 1064 (42000) – syntax error

> The server did not understand the statement: the SQL is not valid – often a reserved word used as a column name, MySQL-8-only syntax on MariaDB, or an unescaped quote.

Example:

```
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'order' at line 1
```

## Common causes

- A reserved word is used as a column or table name without backticks, for example `order`, `key`, `rank` or `groups`.
- The code uses syntax only MySQL 8 knows (`RETURNING`, functional indexes, window functions on old versions) but runs on MariaDB, or the other way around.
- `ANSI_QUOTES` is set in `sql_mode`: double quotes then mean identifiers, not text.
- String building produced broken SQL: a missing comma, an unescaped quote in a value or an empty pair of parentheses.
- The error comes from a migration or an import; the real context is in the line before.

## How to check

Server version and sql_mode:

```bash
mysql -e "SELECT VERSION(), @@sql_mode;"
```

Read the reported statement in the error log with context:

```bash
grep -i -B2 -A2 'syntax' /var/log/mysql/error.log | tail -n 20
```

Compare server and client version:

```bash
mysql -e "SELECT VERSION()"; mysql --version
```

## Fix

1. Quote reserved words with backticks or rename them: `order` becomes `order_ref`. This is the most common cause.
2. Check the target version: what runs on MySQL 8 in staging can fail on MariaDB 10.11. Adjust the query or use the same engine.
3. With `ANSI_QUOTES`, write text values with single quotes and identifiers with backticks.
4. For dynamically built SQL, log the final string before sending it. The missing comma becomes obvious.
5. After the fix run the migration against a copy first, not on production.

## FAQ

**Why does it work on staging but not in production?**

Different versions or `sql_mode`. Compare `SELECT VERSION(), @@sql_mode` on both systems.

**Does an upgrade help?**

Only if the query is valid on the new version. An upgrade does not add missing backticks.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-1064-syntax-error/](https://daniele-michelin.com/en/errors/mysql-1064-syntax-error/)**
