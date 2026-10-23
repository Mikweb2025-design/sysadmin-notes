# Nextcloud: “the database is missing some indexes”

> After an update Nextcloud defined new indexes but did not create them automatically, because that can take long on large tables. Without them queries get slow.

Example:

```
The database is missing some indexes. Due to the fact that adding indexes on big tables could take some time they were not added automatically.
```

## Common causes

- Update to a new major version that brings new indexes.
- Indexes were never added after earlier updates.
- Migration from SQLite or a manual database import without the full schema.

## How to check

Show missing indexes without changing anything:

```bash
sudo -u www-data php /var/www/nextcloud/occ db:add-missing-indices --dry-run
```

Show missing columns and primary keys:

```bash
sudo -u www-data php /var/www/nextcloud/occ db:add-missing-columns --dry-run; sudo -u www-data php /var/www/nextcloud/occ db:add-missing-primary-keys --dry-run
```

Find the largest tables to estimate duration:

```bash
mysql -e "SELECT table_name, ROUND((data_length+index_length)/1024/1024) AS mb FROM information_schema.tables WHERE table_schema='nextcloud' ORDER BY mb DESC LIMIT 5;"
```

## Fix

1. Take a database backup first.
2. On large instances enable maintenance mode: `occ maintenance:mode --on`.
3. Add indexes: `occ db:add-missing-indices`, then `occ db:add-missing-columns` and `occ db:add-missing-primary-keys`.
4. If reported, also run `occ db:convert-filecache-bigint`.
5. Disable maintenance mode and reload the admin overview.

## FAQ

**How long does it take?**

Seconds on small instances. On an oc_filecache with many millions of rows it can take minutes to hours and lock the table meanwhile.

**Can I just ignore the message?**

Nextcloud keeps running, but file lists, shares and search get noticeably slower as data grows. It is worth doing.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-database-is-missing-some-indexes/](https://daniele-michelin.com/en/errors/nextcloud-database-is-missing-some-indexes/)**
