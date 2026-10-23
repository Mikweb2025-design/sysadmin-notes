# OIDC: redirect_uri_mismatch / invalid redirect_uri

> In the authorize request the client sends a redirect_uri that is not registered exactly like that at the identity provider. The IdP stops before the user can log in.

Example:

```
Error 400: redirect_uri_mismatch
```

## Common causes

- http instead of https because the app behind a TLS proxy misdetects its own scheme.
- Trailing slash, `index.php` in the path or a different port.
- Different hostname than registered, e.g. an internal address or www vs. no www.
- New callback URL after an app update while the IdP still has the old one.
- Wrong client: the client ID belongs to another environment (test/production).

## How to check

Get the authorize URL that is sent (Nextcloud user_oidc, provider ID 1):

```bash
curl -s -o /dev/null -w '%{redirect_url}\n' https://cloud.example.com/apps/user_oidc/login/1
```

Decode redirect_uri from the authorize URL:

```bash
python3 -c "import sys,urllib.parse as u;print(u.parse_qs(u.urlparse(sys.argv[1]).query)['redirect_uri'][0])" 'PASTE_AUTHORIZE_URL'
```

Show the Nextcloud proxy settings:

```bash
sudo -u www-data php /var/www/nextcloud/occ config:system:get overwriteprotocol; sudo -u www-data php /var/www/nextcloud/occ config:system:get overwrite.cli.url
```

## Fix

1. Compare the decoded redirect_uri character by character with the IdP registration.
2. Behind a TLS proxy set `'overwriteprotocol' => 'https'` so the app builds https URLs.
3. For Nextcloud user_oidc register `https://cloud.example.com/apps/user_oidc/code`, with `'htaccess.RewriteBase' => '/'` so there is no index.php.
4. No wildcards in production – register exact URLs.
5. In Entra ID the error is AADSTS50011, in Keycloak “Invalid parameter: redirect_uri” – the fix is the same.

## FAQ

**Why does it work internally but not from outside?**

Internally the app builds a URL with an internal hostname or http, but the external https address is registered. The overwrite settings fix that.

**Does a trailing slash really matter?**

Yes. Most IdPs compare the redirect_uri as an exact string. /code and /code/ are two different URLs.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/oidc-redirect-uri-mismatch-invalid-redirect-uri/](https://daniele-michelin.com/en/errors/oidc-redirect-uri-mismatch-invalid-redirect-uri/)**
