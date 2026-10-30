# nginx: [emerg] could not build server_names_hash, you should increase server_names_hash_bucket_size

> nginx stores all `server_name` values in a hash table. A very long name or many names no longer fit into one bucket, so the configuration test fails and nginx will not start or reload.

Example:

```
nginx: [emerg] could not build server_names_hash, you should increase server_names_hash_bucket_size: 64
nginx: configuration file /etc/nginx/nginx.conf test failed
```

## Common causes

- One `server_name` is very long (for example a long subdomain), and a bucket is too small for it.
- Many server names were added across `server` blocks, so the table needs a larger size.
- The same name is listed in several places because a file is included twice, which inflates the table.
- A generated config (wildcard lists, many aliases per site) was added without raising the hash settings.

## How to check

Run the configuration test:

```bash
nginx -t
```

Current hash settings, if any:

```bash
nginx -T 2>/dev/null | grep -nE 'server_names_hash'
```

The longest configured server names:

```bash
nginx -T 2>/dev/null | grep -E '^\s*server_name' | tr -s ' ;' '\n' | awk 'length > 25' | sort -u
```

Is a file included twice?:

```bash
nginx -T 2>/dev/null | grep -E '^# configuration file' | sort | uniq -d
```

## Fix

1. Add to the `http` block in `/etc/nginx/nginx.conf`: `server_names_hash_bucket_size 128;` (a power of two; the message names the current value).
2. If the error stays or you have hundreds of names, also raise `server_names_hash_max_size`, for example `1024`.
3. Remove duplicate includes so no name is defined twice.
4. Test and apply: `nginx -t && systemctl reload nginx`.

## FAQ

**Which value should I use?**

Double the value from the message and try again: 64 becomes 128, then 256 if needed. The bucket size must be a power of two.

**Why does it appear after adding one more domain?**

The table is sized when the configuration loads. One more or one longer name can be the one that no longer fits, even though the old setup worked.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nginx-could-not-build-server-names-hash/](https://daniele-michelin.com/en/errors/nginx-could-not-build-server-names-hash/)**
