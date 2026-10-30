# bash: /usr/bin/rm: Argument list too long

> The shell replaced `*` with every matching file name, and the list got larger than the kernel allows for one command. The command never started. It is a limit of the call, not of the disk.

Example:

```
$ rm /var/spool/app/cache/*.tmp
-bash: /usr/bin/rm: Argument list too long
```

## Common causes

- A directory holds a very large number of files (cache, sessions, mail queue, uploads) and the pattern matches them all.
- The limit `ARG_MAX` for arguments plus environment of one program call is exceeded.
- Very long file names make the total size reach the limit with fewer files.
- A huge environment (many or large variables) leaves less room for the arguments.

## How to check

What is the limit on this system?:

```bash
getconf ARG_MAX
```

How many entries are in the directory? (ls -f does not sort, so it is fast):

```bash
ls -f /var/spool/app/cache | wc -l
```

What would the pattern match? Print first, do not delete yet:

```bash
find /var/spool/app/cache -maxdepth 1 -type f -name '*.tmp' -print | head -n 20
```

Stack limit that the argument limit is based on:

```bash
ulimit -s
```

## Fix

1. Let `find` do the matching instead of the shell. First look at the result with `-print`, then delete: `find /var/spool/app/cache -maxdepth 1 -type f -name '*.tmp' -delete`. This removes the files for good.
2. Run a command in batches: `find DIR -maxdepth 1 -type f -name '*.log' -exec gzip {} +`.
3. With xargs and safe names: `find DIR -maxdepth 1 -type f -name '*.jpg' -print0 | xargs -0 -n 500 mv -t /target`.
4. Narrow the pattern to a part of the files, for example `rm 2024-01-*`, only as a workaround.
5. Find out why so many files pile up (cleanup job, session handling, logrotate) and fix that.

## FAQ

**Does raising ulimit -s help?**

It can raise the limit, but it is a workaround. `find -exec` or `xargs` work with any number of files.

**Why is rm -rf on the directory not affected?**

Without a `*` the shell passes one name only. `rm -rf DIR` deletes the whole directory, so use it only if you really want all of it gone.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/bash-argument-list-too-long/](https://daniele-michelin.com/en/errors/bash-argument-list-too-long/)**
