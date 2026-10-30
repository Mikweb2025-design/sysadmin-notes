# SAML: Invalid audience for this Response (AudienceRestriction)

> The identity provider addresses its assertion to one audience, the Entity ID of the service provider. Your app checks that this value equals its own Entity ID and refuses otherwise. Signature and time checks are separate, so this is purely a name mismatch.

Example:

```
OneLogin_Saml2_ValidationError: Invalid audience for this Response (expected 'https://cloud.example.com/apps/user_saml/saml/metadata', got 'https://cloud.example.com')
```

## Common causes

- The Entity ID (audience) configured in the IdP differs from the one in the service provider, often by a trailing slash, `http` against `https`, or the metadata URL against a plain name.
- The IdP application was created for another app or environment, for example test against production, and keeps the old value.
- The service provider builds its Entity ID from the request URL, and behind a reverse proxy that URL differs from the public one.
- The IdP metadata was imported before the Entity ID was changed and never refreshed.

## How to check

What audience does the response carry? (paste the decoded response in a file):

```bash
base64 -d saml-response.b64 | xmllint --format - | grep -i -A2 Audience
```

What Entity ID does the service provider publish?:

```bash
curl -s https://cloud.example.com/index.php/apps/user_saml/saml/metadata | grep -o 'entityID="[^"]*"'
```

What is stored in the Nextcloud SAML settings?:

```bash
sudo -u www-data php occ saml:config:get | grep -i entityid
```

Does Nextcloud know its public protocol?:

```bash
sudo -u www-data php occ config:system:get overwriteprotocol
```

## Fix

1. Pick one exact Entity ID and make both sides identical, character for character, including case and the trailing slash.
2. Enter that value as the audience or Entity ID of the application in the IdP, and as the service provider Entity ID in the app.
3. Behind a proxy, set `occ config:system:set overwriteprotocol --value=https` and the right host, so the app builds the public address.
4. Re-import or re-save the IdP metadata and the service provider metadata after any change.
5. Log in again in a private window and decode a new response to confirm that the audience now matches.

## FAQ

**Is this the same as a clock skew error?**

No. A clock error shows `NotBefore` or `NotOnOrAfter` and is about time. Invalid audience is about names only, and clocks can be perfect.

**Should the Entity ID be the metadata URL?**

It can be, if both sides use the same. Many IdPs prefill the URL of the metadata, others use a plain name or a URN. What counts is that both sides carry the same string.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/saml-invalid-audience-for-this-response/](https://daniele-michelin.com/en/errors/saml-invalid-audience-for-this-response/)**
