# ERROR 1067 (42000): Invalid default value for …

> The server rejects a column default: it is invalid for the type or conflicts with the active `sql_mode`, for example a zero date under `NO_ZERO_DATE`.

Example:

```
ERROR 1067 (42000): Invalid default value for 'created_at'
```

## Common causes

- A `TIMESTAMP` or `DATETIME` column defaults to `'0000-00-00 00:00:00'` while `NO_ZERO_DATE` is on.
- The dump comes from MySQL 5.6 and uses defaults that 5.7/8.0 reject under `STRICT_TRANS_TABLES`.
- The default does not match the type, for example text on a numeric column or a default on `TEXT` before MySQL 8.0.13.
- The character set or collation of the default does not match the column, common in old dumps.
- A migration puts `CURRENT_TIMESTAMP` on a column that does not allow it in this version.

## How to check

Show the active sql_mode:

```bash
mysql -e "SELECT @@sql_mode;"
```

Inspect the column definition:

```bash
mysqldump --no-data db table | grep -i -A3 -E 'timestamp|datetime'
```

Compare source and target version:

```bash
mysql --version; grep -i -m1 'server version' dump.sql
```

## Fix

1. Change the default to match the type: `DEFAULT CURRENT_TIMESTAMP` instead of a zero date, or allow `NULL`.
2. For an import, relax the mode for the session only: `SET SESSION sql_mode='NO_ENGINE_SUBSTITUTION';` – deliberately and not permanently.
3. Better: test the dump on the target version and regenerate it with suitable options instead of bending the mode.
4. Check `sql_mode` again after the import, otherwise zero dates slip in silently.

## FAQ

**Why does this only appear after a server upgrade?**

Because `STRICT_TRANS_TABLES` and `NO_ZERO_DATE` are on by default in newer versions.

**Can I turn sql_mode off permanently?**

Technically yes, but then the database silently stores wrong values. Only useful temporarily during an import.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mysql-1067-invalid-default-value/](https://daniele-michelin.com/en/errors/mysql-1067-invalid-default-value/)**
