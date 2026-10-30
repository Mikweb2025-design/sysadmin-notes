# DNS: connection timed out; no servers could be reached

> The query got no answer at all, not even a refusal. The packet never arrived or the reply never came back. This points to the path or the server process, not to the domain.

Example:

```
$ dig example.com
;; communications error to 192.0.2.53#53: timed out
;; communications error to 192.0.2.53#53: timed out
;; no servers could be reached
```

## Common causes

- The configured DNS server address is wrong or the server is down.
- A firewall drops UDP and TCP port 53, on the client, the server or in between.
- The DNS service (`named`, `unbound`, `dnsmasq`) does not run or listens only on another interface.
- No route to the server, for example a VPN or network change.
- Large answers need TCP, and only UDP is allowed.

## How to check

Which server does the system use?:

```bash
resolvectl status
```

Same query against a public resolver. Does the network work at all?:

```bash
dig example.com @1.1.1.1 +time=3 +tries=1
```

Query over TCP instead of UDP:

```bash
dig example.com @192.0.2.53 +tcp +time=3 +tries=1
```

On the DNS server: does it listen on port 53?:

```bash
sudo ss -lunpt | grep ':53 '
```

## Fix

1. If the query to a public resolver works, the configured server is the problem: replace it or repair it.
2. If nothing works, fix the network first: route, gateway, VPN, cable.
3. On your own DNS server, start the service (`sudo systemctl status unbound` or `named`) and check that it listens on the interface the clients use.
4. Allow port 53 for UDP and TCP in the firewall in both directions as needed. Allow it only from the networks that need it.
5. If only TCP fails, answers may be big or cut. Allow TCP 53, do not block it.

## FAQ

**Is the domain broken?**

Probably not. No answer means the resolver or the path is broken. A broken domain gives an answer such as NXDOMAIN or SERVFAIL.

**It works with 8.8.8.8 and not with my router.**

Then the router or the upstream behind it is the problem. Set another resolver on the client to confirm, then fix or replace the router DNS.

---
More detail, related guides and tools: **[daniele-michelin.com/en/errors/dig-connection-timed-out-no-servers-could-be-reached/](https://daniele-michelin.com/en/errors/dig-connection-timed-out-no-servers-could-be-reached/)**
