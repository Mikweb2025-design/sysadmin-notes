# PHP Warning: Cannot modify header information - headers already sent by (output started at ...)

> PHP sends HTTP headers (redirect, cookie, session) only before any body output. Something printed text first, even a single space or a hidden byte. The message names the file and line where the output started.

Example:

```
PHP Warning:  Cannot modify header information - headers already sent by (output started at /var/www/example.com/config.php:1) in /var/www/example.com/login.php on line 12
```

## Common causes

- A UTF-8 BOM (hidden bytes) at the start of a PHP file, often saved by an editor, usually in `config.php` or a template.
- Whitespace or a blank line before `` in an included file.
- `echo`, `print` or HTML output before `header()`, `setcookie()` or `session_start()`.
- A PHP warning or notice printed to the page before the header call (`display_errors` on), which then causes this second error.
- A plugin or theme in a CMS that prints output early.

## How to check

Find the file named in the message and check its first bytes for a BOM:

```bash
head -c 3 /var/www/example.com/config.php | xxd
```

Search PHP files for a BOM:

```bash
grep -rl $'\xEF\xBB\xBF' --include='*.php' /var/www/example.com | head
```

Is there output after a closing tag at the end of the file?:

```bash
tail -c 30 /var/www/example.com/config.php | xxd
```

Check the file for syntax problems:

```bash
php -l /var/www/example.com/config.php
```

## Fix

1. Open the file the message names and remove everything before `<?php`, including the BOM (save as UTF-8 without BOM).
2. In files that contain only PHP, leave out the closing `?>` so no trailing newline is sent.
3. Move `header()`, `setcookie()` and `session_start()` above any output, or restructure so output happens last.
4. If an earlier warning is the first output, fix it or set `display_errors = Off` in production and log errors instead.
5. As a stopgap only, `ob_start();` at the top buffers output, but fix the real cause.

## FAQ

**What does output started at config.php:1 mean?**

The first byte was sent from line 1 of that file. That points to a BOM or whitespace before the opening tag.

**Is it safe to ignore the warning?**

No. The redirect, cookie or session that should have been set is lost, so logins and redirects can silently fail.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-cannot-modify-header-information-headers-already-sent/](https://daniele-michelin.com/en/errors/php-cannot-modify-header-information-headers-already-sent/)**
