# OIDC/OAuth2: invalid_grant at the token endpoint

> The identity provider rejects the authorization code or refresh token the client wants to exchange. The login fails right after the redirect back from the IdP.

Example:

```
{"error":"invalid_grant","error_description":"Code not valid"}
```

## Common causes

- Authorization code expired (often after 30–60 seconds) or already redeemed, e.g. by a double click or browser reload.
- redirect_uri in the token request differs from the one in the authorize request.
- Refresh token expired, revoked or the IdP session ended (Keycloak: “Session not active”).
- PKCE: the code_verifier does not match the code_challenge.
- Client and IdP clocks differ significantly.

## How to check

Check time synchronisation:

```bash
timedatectl | grep -E 'synchronized|Local time'
```

Search client or IdP logs for the error:

```bash
journalctl --since '15 min ago' | grep -iE 'invalid_grant|token endpoint|oidc'
```

Show the IdP's issuer and token endpoint:

```bash
curl -s https://sso.example.com/realms/main/.well-known/openid-configuration | jq -r '.issuer, .token_endpoint'
```

## Fix

1. Restart the login cleanly, without the back button or a reload on the callback URL.
2. Keep redirect_uri identical in app and IdP, including scheme and trailing slash.
3. Sync the clock via NTP: `timedatectl set-ntp true`.
4. For refresh errors align session lifetimes on the IdP with the app's token lifetime.
5. Several app instances behind a load balancer: use shared session storage so the PKCE verifier is available on every instance.

## FAQ

**What does “Code not valid” mean in Keycloak?**

The code was already redeemed, has expired or belongs to another session. Often a second callback request from a reload or prefetch is the cause.

**Does the user need to do anything?**

Usually a fresh login is enough. If it keeps happening, the problem is in the app, proxy or IdP configuration.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/oidc-invalid-grant-token-endpoint/](https://daniele-michelin.com/en/errors/oidc-invalid-grant-token-endpoint/)**
