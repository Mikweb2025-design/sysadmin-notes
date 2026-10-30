# OIDC: invalid_client (client authentication failed)

> After the user signs in, your app calls the token endpoint and has to prove who it is with its client ID and secret. `invalid_client` means that proof was refused. It is about the app's identity, not the user's code or redirect address.

Example:

```
{"error":"invalid_client","error_description":"Client authentication failed"}
HTTP/1.1 401 Unauthorized
```

## Common causes

- The client secret is wrong, was rotated, has expired, or was copied with a space or line break.
- The client uses another authentication method than the IdP expects, for example `client_secret_post` against `client_secret_basic`.
- The client ID is mistyped, belongs to another realm or tenant, or the client was deleted or disabled.
- The client is set as a public client without a secret, but the app sends one, or it is confidential and sends none.
- A proxy or a form filler changes the `Authorization` header.

## How to check

Which auth methods does the IdP accept?:

```bash
curl -s https://idp.example.com/.well-known/openid-configuration | jq '.token_endpoint, .token_endpoint_auth_methods_supported'
```

Which client ID is configured in Nextcloud?:

```bash
sudo -u www-data php occ user_oidc:provider
```

Find the error in the app log:

```bash
grep -i invalid_client /var/www/nextcloud/data/nextcloud.log | tail -n 5
```

Hidden characters in the stored secret value?:

```bash
printf '%s' "$CLIENT_SECRET" | od -c | tail -n 3
```

## Fix

1. Open the client in the IdP, generate or copy the secret again, and paste it without spaces or line breaks into the app.
2. Check the expiry date of the secret. Some providers, for example Microsoft Entra ID, let secrets expire, so create a new one and update the app before the old one stops.
3. Match the authentication method: set the client in the IdP and the app to the same one, `client_secret_basic` or `client_secret_post`.
4. Check that the client is enabled, is of type confidential when it uses a secret, and belongs to the realm or tenant of the issuer URL you use.
5. Try the login again and read the IdP's own log for the exact reason.

## FAQ

**What is the difference to invalid_grant?**

invalid_client means the app could not prove its identity. `invalid_grant` means the code or token was bad: used twice, expired, or issued to another redirect address.

**Is a wrong user password the cause?**

No. The user is already signed in at this point. The failure is between your app and the IdP, so changing the user's password will not help.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/oidc-invalid-client-client-authentication-failed/](https://daniele-michelin.com/en/errors/oidc-invalid-client-client-authentication-failed/)**
