# SAML: “not yet valid” – clock skew on NotBefore

> Every SAML assertion is only valid within a time window (NotBefore to NotOnOrAfter). If the service provider's clock differs from the IdP's, the assertion counts as not yet or no longer valid.

Example:

```
Could not validate timestamp: not yet valid. Check system clock.
```

## Common causes

- Server clock without NTP runs fast or slow, typical for VMs after a snapshot restore.
- The NTP service runs but cannot reach a time server (firewall on UDP 123).
- The IdP server (e.g. ADFS) has the wrong time itself.
- Wrong time zone confused with wrong time – SAML always uses UTC.
- Very short assertion lifetime at the IdP combined with a slow redirect.

## How to check

Show time and NTP status:

```bash
timedatectl status
```

Show the offset to the time server:

```bash
chronyc tracking 2>/dev/null || ntpq -p
```

Compare UTC time of SP and IdP directly:

```bash
date -u; ssh idp.example.com date -u
```

## Fix

1. Enable NTP: `timedatectl set-ntp true`, or install and start chrony.
2. Allow outbound UDP 123 to the time servers.
3. Check the time on the IdP as well; with ADFS set `NotBeforeSkew` to 1–2 minutes if needed.
4. Do not raise tolerances to many minutes – that weakens protection against replayed assertions.
5. On VMs make sure time is synced via the hypervisor or NTP.

## FAQ

**How much skew is allowed?**

Most libraries tolerate a few seconds up to about 3 minutes. With working NTP the offset stays below one second.

**Why does it only happen sometimes?**

Because the clock drifts slowly. Right after a correction it works, hours later it does not. Only a running NTP service fixes it for good.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/saml-clock-skew-notbefore-timestamp-not-yet-valid/](https://daniele-michelin.com/en/errors/saml-clock-skew-notbefore-timestamp-not-yet-valid/)**
