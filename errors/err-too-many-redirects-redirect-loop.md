# ERR_TOO_MANY_REDIRECTS: example.com redirected you too many times

> Two places keep sending the browser to each other. The browser follows the redirects, gives up after a limit and shows this error. The loop is always in the combination of server, CDN and application.

Example:

```
This page isn't working
example.com redirected you too many times.
ERR_TOO_MANY_REDIRECTS
```

## Common causes

- A CDN or proxy talks plain HTTP to your server, while the server redirects every HTTP request to HTTPS (a typical *Flexible SSL* loop).
- The application builds the redirect from the wrong scheme because the proxy does not pass `X-Forwarded-Proto`, so it redirects to HTTPS again and again.
- Conflicting www and non-www rules: the server redirects to one, the application or CMS to the other.
- A CMS has `siteurl` or `home` set to a different address than the one visitors use.
- Stale cookies in a login redirect loop (works in a private window).

## How to check

Follow the redirects with the limit and read each Location:

```bash
curl -sIL --max-redirs 10 https://example.com/ | grep -iE '^(HTTP|location)'
```

What does plain HTTP do?:

```bash
curl -sI http://example.com/ | grep -iE '^(HTTP|location)'
```

Redirect rules in the nginx config:

```bash
nginx -T 2>/dev/null | grep -nE 'return 30[0-9]|rewrite .*(permanent|redirect)|X-Forwarded-Proto'
```

Test with the proxy header the app expects:

```bash
curl -sI -H 'X-Forwarded-Proto: https' http://127.0.0.1/ -H 'Host: example.com' | grep -iE '^(HTTP|location)'
```

## Fix

1. Read the Location chain from the first check: the two URLs that alternate show who redirects where.
2. With a CDN, set its SSL mode so it connects to your origin over HTTPS (for example *Full*), not plain HTTP.
3. Behind a proxy pass the scheme: `proxy_set_header X-Forwarded-Proto $scheme;`, and make the application trust it.
4. Decide on one canonical host (www or not) and redirect only once, in one place.
5. Fix `siteurl` and `home` of the CMS, clear the site cookies, and test again.

## FAQ

**Why does it work in a private window?**

A cookie, often a session or login cookie, takes part in the loop. Clearing cookies for the site confirms it.

**How many redirects does the browser follow?**

A limit exists and differs per browser, so do not rely on a number. Any loop eventually ends in this error.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/err-too-many-redirects-redirect-loop/](https://daniele-michelin.com/en/errors/err-too-many-redirects-redirect-loop/)**
