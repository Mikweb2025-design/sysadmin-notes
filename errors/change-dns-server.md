# Changing the DNS server: when, where and the risks

> DNS turns names like example.com into addresses. If your provider's DNS server stops answering, websites seem dead even though the line works. A different DNS server can get around it.

Example:

```
Webseiten laden nicht, obwohl das WLAN verbunden ist; ping 8.8.8.8 geht, aber nslookup example.com scheitert.
```

## Common causes

- The provider's DNS server or the router's DNS service is faulty or overloaded.
- A wrong or outdated DNS entry is set in the router or on your computer.
- A filter or ad blocker in the network blocks the name you want to open.
- A cached (old) entry returns a wrong address.

## How to check

Does a plain address work? If yes, DNS is the likely cause (Windows: `ping -n 4 …`):

```bash
ping -c 4 8.8.8.8
```

Does the router's DNS answer?:

```bash
nslookup example.com
```

Does another DNS server (Google 8.8.8.8) answer?:

```bash
nslookup example.com 8.8.8.8
```

Windows: clear the DNS cache:

```bash
ipconfig /flushdns
```

## Fix

1. Run the tests above. If `nslookup example.com 8.8.8.8` works but `nslookup example.com` does not, your current DNS server is the problem.
2. The best place to change it is the router: in its menu under home network or internet, enter the DNS server, and it applies to every device. Alternatively change it on a single PC in the network settings.
3. Well-known public DNS servers are Cloudflare 1.1.1.1, Google 8.8.8.8 and Quad9 9.9.9.9. Enter one as preferred and a second as alternate.
4. Mind your privacy: the DNS provider you pick can see which names you look up. Read its privacy policy and decide on purpose.
5. Then clear the cache (`ipconfig /flushdns` on Windows) and verify with `nslookup example.com`.
6. If all works again, you may report the fault to your provider or keep the new DNS server.

## FAQ

**Is a public DNS server faster or safer?**

Not automatically. Sometimes it answers faster, sometimes not, and you hand your lookups to another operator. Switch mainly when your current server is causing trouble.

**Will this make my connection slower?**

It only changes name lookup, not the line. The download itself does not get faster or slower because of it.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/change-dns-server/](https://daniele-michelin.com/en/errors/change-dns-server/)**
