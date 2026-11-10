# tar/gzip: Unexpected EOF in archive – unexpected end of file

> The archive ends in the middle of the data. Bytes are missing, so tar cannot read the last member and aborts.

Example:

```
gzip: stdin: unexpected end of file
tar: Unexpected EOF in archive
tar: Error is not recoverable: exiting now
```

## Common causes

- A download or copy was interrupted and the file is shorter than announced.
- The disk filled up during creation or the process was killed.
- A transfer in ASCII mode changed bytes, for example over FTP.
- The archive was read while still being written, or sits on a filesystem with errors.
- The file was modified after creation, for example by an incomplete `scp`.

## How to check

Test the compression:

```bash
gzip -t backup.tar.gz; echo "exit=$?"
```

Check the archive without extracting:

```bash
tar -tzf backup.tar.gz > /dev/null; echo "exit=$?"
```

Size and checksum:

```bash
ls -l backup.tar.gz; sha256sum backup.tar.gz
```

How far can it be read?:

```bash
tar -tzf backup.tar.gz 2>/dev/null | tail -n 5
```

## Fix

1. Download or recreate it while the source still exists. That is the clean path.
2. For a partial archive, salvage the readable parts: `tar -xzf backup.tar.gz --ignore-zeros` or `gzip -dc backup.tar.gz | tar -x`.
3. Use `gzip -dc | tar -tv` to see how far the data is readable.
4. Store a checksum after creating archives and compare it before restoring.
5. Do not write backups to the same filesystem as the source, and check the exit code of the creation step.

## FAQ

**Can I repair a broken gz?**

Not fully. Readable parts can be extracted, but the missing bytes are gone.

**Why is the archive smaller than expected?**

The write was interrupted: full disk, killed process or an aborted transfer.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/tar-unexpected-end-of-file/](https://daniele-michelin.com/en/errors/tar-unexpected-end-of-file/)**
