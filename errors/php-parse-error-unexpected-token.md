# PHP Parse error: syntax error, unexpected token

> PHP could not compile the file, so it never runs. The reported line is often just the consequence of an error a few lines earlier.

Example:

```
PHP Parse error: syntax error, unexpected token ";" in /var/www/html/wp-content/plugins/x/functions.php on line 214
```

## Common causes

- Syntax removed in PHP 8: string offsets with curly braces `$str{0}`, `create_function` or other old constructs.
- The file is half uploaded or truncated, for example after an interrupted FTP or deploy.
- A semicolon, brace or quote is missing in the line before.
- Invisible characters from the editor: a BOM at the start of the file or non-breaking spaces.
- A plugin is parsed by a newer PHP version than it supports.

## How to check

Lint the reported file:

```bash
php -l /var/www/html/wp-content/plugins/x/functions.php
```

PHP version and recent errors:

```bash
php -v; tail -n 20 /var/log/php8.2-fpm.log
```

Find old string offset syntax:

```bash
grep -rnE '\$[a-zA-Z_]+\{[0-9]+\}' /var/www/html --include='*.php' | head
```

Find BOM and non-breaking spaces:

```bash
grep -rlP '^\xEF\xBB\xBF' /var/www/html --include='*.php' | head
```

## Fix

1. Run `php -l` on the reported file first: the line in the log is often not the real cause.
2. When moving to PHP 8, replace removed syntax: `$str{0}` becomes `$str[0]`, `create_function` becomes a closure.
3. Re-upload truncated files or replay the update if the upload was incomplete.
4. Remove BOM and non-breaking spaces and save the file as UTF-8 without BOM.
5. Clear the opcache afterwards, otherwise the old broken version keeps running.

## FAQ

**Why does PHP point at a line that looks fine?**

Usually a semicolon or brace is missing before it. Check two or three lines above the message.

**The error started right after a PHP upgrade – should I roll back?**

As an emergency measure yes, but better to find and replace the removed syntax.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/php-parse-error-unexpected-token/](https://daniele-michelin.com/en/errors/php-parse-error-unexpected-token/)**
