# NET::ERR_CERT_DATE_INVALID: certificate date is invalid

> A certificate is only valid between two dates. The browser compares them with its own clock. The error means that today, as seen by the client, lies outside that window, so either the certificate is out of date or the client's clock is wrong.

Example:

```
Your clock is ahead
NET::ERR_CERT_DATE_INVALID
This server could not prove that it is cloud.example.com; its security certificate expired or is not yet valid.
```

## Common causes

- The clock on the device is wrong: empty CMOS battery, a VM restored from a snapshot, a device without network time, or a wrong time zone or year.
- The server certificate really expired and was not renewed.
- The certificate is not yet valid (`notBefore` in the future), often because the machine that issued it had a wrong clock.
- A certificate in the chain, such as an intermediate, expired even though yours is fine.

## How to check

Validity dates of the served certificate:

```bash
echo | openssl s_client -connect cloud.example.com:443 -servername cloud.example.com 2>/dev/null | openssl x509 -noout -dates
```

What time does this machine think it is?:

```bash
date -u
```

Is network time active?:

```bash
timedatectl status
```

What does the server's own clock say?:

```bash
curl -sI https://cloud.example.com | grep -i '^date:'
```

## Fix

1. Compare `notBefore` and `notAfter` with the current date. If today is inside the window on a correct clock, the problem is the client clock.
2. On the client, switch on network time: `sudo timedatectl set-ntp true`, and fix the time zone. On phones and PCs use the automatic date and time setting.
3. In virtual machines, enable time sync with the host or run an NTP client, and correct the time after restoring a snapshot.
4. If the certificate itself is expired, renew it and reload the web server. See the expired-certificate entry for the steps.
5. If it is not yet valid, fix the clock of the issuing machine and issue the certificate again.

## FAQ

**Why do only some users see this error?**

Those users have a wrong device clock. If everyone sees it, the certificate or the server clock is the cause.

**Why do other sites also fail on the same computer?**

All HTTPS sites are checked against the same wrong clock, so a wrong date makes many sites fail at once. That is the clearest sign of a clock problem.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/net-err-cert-date-invalid-clock-wrong/](https://daniele-michelin.com/en/errors/net-err-cert-date-invalid-clock-wrong/)**
