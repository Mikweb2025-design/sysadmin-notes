# SAML error: signature validation failed in Nextcloud

> Nextcloud (user_saml with php-saml) could not verify the SAML response signature with the stored IdP certificate and rejects the login.

Example:

```
OneLogin\Saml2\ValidationError: Signature validation failed. SAML Response rejected
```

## Common causes

- The identity provider (Entra ID, ADFS, Keycloak) rotated its signing certificate and Nextcloud still has the old one.
- The wrong certificate is stored, e.g. the encryption instead of the signing certificate.
- Signature settings do not match: Nextcloud expects signed assertions while the IdP only signs the response (or vice versa).
- The certificate was damaged while copying (line breaks, missing characters).

## How to check

Show the SAML configuration in Nextcloud:

```bash
sudo -u www-data php /var/www/nextcloud/occ saml:config:get
```

Read the current IdP certificates from the metadata:

```bash
curl -s https://sso.example.com/realms/main/protocol/saml/descriptor | grep -o '<ds:X509Certificate>[^<]*'
```

Fingerprint and expiry of the stored certificate:

```bash
openssl x509 -in idp.crt -noout -fingerprint -sha256 -enddate
```

Find SAML errors in the Nextcloud log:

```bash
grep -i 'saml' /var/www/nextcloud/data/nextcloud.log | tail -n 5
```

## Fix

1. Paste the current signing certificate from the IdP metadata into Nextcloud's SSO settings (Base64 block only, no blank lines).
2. Align signature options with the IdP, e.g. signed assertions vs. signed response.
3. For planned rotations, add the new certificate in time if the IdP uses both in parallel.
4. For debugging, set `loglevel` to 0 briefly and revert afterwards.

## FAQ

**Is this about the website's TLS certificate?**

No. SAML uses a separate signing certificate from the IdP. Nextcloud's HTTPS certificate plays no role here.

**Why did it work yesterday?**

Almost always the IdP certificate was renewed. Entra ID and ADFS rotate automatically; compare the fingerprint in the metadata with the one in Nextcloud.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/nextcloud-saml-signature-validation-failed/](https://daniele-michelin.com/en/errors/nextcloud-saml-signature-validation-failed/)**
