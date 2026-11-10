# rsync error: some files/attrs were not transferred (code 23)

> rsync finished the run but did not copy everything. Code 23 is a summary – the lines above it name the file and the reason.

Example:

```
rsync: [sender] send_files failed to open "/var/log/app/current.log": Permission denied (13)
rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1338)
```

## Common causes

- The rsync user cannot read source files or directories.
- Files change during the run, such as logs, caches or database files.
- Attributes like ACLs or xattrs cannot be applied on the target.
- The target filesystem cannot store permissions or owners, for example FAT or NFS with `root_squash`.
- A file disappears between the file list and the transfer, common with rotated logs.

## How to check

Exit code and statistics:

```bash
rsync -a --stats src/ dst/ 2>&1 | tail -n 15; echo "exit=$?"
```

Show only the error lines:

```bash
rsync -a src/ dst/ 2>&1 | grep -E 'rsync: |failed|Permission denied' | head -n 20
```

Dry run without writing:

```bash
rsync -an --itemize-changes src/ dst/ | head -n 20
```

Check permissions along the path:

```bash
namei -l /backup/data
```

## Fix

1. Read the individual error lines: code 23 is the summary, the reason is right above it.
2. Check read permissions and run rsync as the right user instead of forcing root.
3. For changing files, take a consistent database backup first or skip directories with `--exclude`.
4. Only transfer attributes the target supports: `-a` without `-AX`, or use a suitable filesystem.
5. For backups, 23 is a failure even if almost everything was copied. Do not treat it as success.

## FAQ

**Can I ignore code 23?**

Only if you know exactly which files are missing. `--itemize-changes` and the error lines show it; for backups 23 is a failure.

**Why does it only happen on the second run?**

Because files changed during the run or attributes could not be set on the target.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/rsync-code-23-partial-transfer/](https://daniele-michelin.com/en/errors/rsync-code-23-partial-transfer/)**
