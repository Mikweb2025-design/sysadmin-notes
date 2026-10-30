# Mixed Content: The page was loaded over HTTPS, but requested an insecure resource ... This request has been blocked

> The page itself uses HTTPS, but it tries to load a script, stylesheet, font, image or request over plain HTTP. Browsers block active content (scripts, styles, requests) and warn or upgrade passive content such as images. Styles and scripts then fail to load.

Example:

```
Mixed Content: The page at 'https://example.com/' was loaded over HTTPS, but requested an insecure script 'http://example.com/js/app.js'. This request has been blocked; the content must be served over HTTPS.
```

## Common causes

- Hard-coded `http://` URLs in templates, CSS, JavaScript or in the content stored in the database.
- The site moved to HTTPS but the CMS still has `http://` in `siteurl`, `home` or in old posts.
- Behind a reverse proxy the application does not know the request was HTTPS and generates `http://` links (missing `X-Forwarded-Proto`).
- A third-party script, font or widget is loaded over HTTP.
- A redirect inside the application sends an API or asset request to an `http://` address.

## How to check

List http:// resources in the delivered HTML:

```bash
curl -s https://example.com/ | grep -oE '(src|href|action)="http://[^"]+"' | sort -u | head -n 20
```

Is a policy that upgrades requests already set?:

```bash
curl -sI https://example.com/ | grep -i 'content-security-policy'
```

Does the proxy pass the scheme to the app?:

```bash
nginx -T 2>/dev/null | grep -n 'X-Forwarded-Proto'
```

The exact blocked URLs:

```bash
curl -s https://example.com/assets/app.css | grep -oE 'url\(http://[^)]+\)' | sort -u
```

## Fix

1. Replace `http://` with `https://` (or a relative URL such as `/js/app.js`) in templates, CSS and JavaScript.
2. Update the CMS: set the site address to `https://` and run its search-and-replace for old content in the database, with a backup first.
3. Pass the scheme behind a proxy: `proxy_set_header X-Forwarded-Proto $scheme;`, and tell the application to trust it.
4. As a stopgap, send `Content-Security-Policy: upgrade-insecure-requests` so the browser upgrades requests that the server supports over HTTPS.
5. Replace third-party resources that have no HTTPS version.

## FAQ

**Why are some mixed resources only warned and others blocked?**

Browsers treat scripts, styles and requests (active content) as blocked, and images, audio and video (passive content) more leniently, or upgrade them. Rely on neither: fix all of them.

**Is upgrade-insecure-requests a real fix?**

It works only when the target also exists on HTTPS. It is a good safety net, but the URLs should still be corrected at the source.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/mixed-content-blocked-insecure-resource/](https://daniele-michelin.com/en/errors/mixed-content-blocked-insecure-resource/)**
