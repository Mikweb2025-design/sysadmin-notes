# Sysadmin glossary

## [1st, 2nd and 3rd level support](https://daniele-michelin.com/en/glossary/support-levels-1st-2nd-3rd/)

1st, 2nd and 3rd level describe the tiers of IT support, from taking requests and solving standard cases to deep technical analysis and fixing bugs in code.

## [A and AAAA record](https://daniele-michelin.com/en/glossary/a-aaaa-record/)

An A record maps a hostname to an IPv4 address; an AAAA record maps it to an IPv6 address.

```
dig +short A cloud.example.com; dig +short AAAA cloud.example.com
```

## [ACME](https://daniele-michelin.com/en/glossary/acme/)

ACME (Automatic Certificate Management Environment) is the protocol for requesting, validating and renewing certificates automatically, best known from Let's Encrypt.

```
certbot renew --dry-run
```

## [Authoritative name server](https://daniele-michelin.com/en/glossary/authoritative-name-server/)

An authoritative name server stores a zone's records itself and answers queries about them definitively, without caching.

```
dig @ns1.example-dns.net example.com A +norecurse
```

## [Bounce / NDR](https://daniele-michelin.com/en/glossary/bounce-ndr/)

A bounce or NDR (non-delivery report) is an automatic message from a mail server saying an email could not be delivered.

```
550 5.1.1 <max@example.com>: Recipient address rejected: User unknown in virtual mailbox table
```

## [CAA record](https://daniele-michelin.com/en/glossary/caa-record/)

A CAA record states in DNS which certificate authorities (CAs) are allowed to issue TLS certificates for a domain.

```
example.com. 3600 IN CAA 0 issue "letsencrypt.org"
```

## [CalDAV and CardDAV](https://daniele-michelin.com/en/glossary/caldav-carddav/)

CalDAV and CardDAV are open protocols built on WebDAV that synchronise calendars (CalDAV) and address books (CardDAV) between a server and devices.

```
curl -sI https://cloud.example.com/.well-known/caldav
```

## [Certificate chain](https://daniele-michelin.com/en/glossary/certificate-chain/)

A certificate chain is the sequence of server certificate, one or more intermediate certificates and the root CA that a client uses to verify authenticity.

```
openssl s_client -connect example.com:443 -servername example.com -showcerts
```

## [CNAME record](https://daniele-michelin.com/en/glossary/cname-record/)

A CNAME record declares a DNS name to be an alias of another name, whose records then apply.

```
www.example.com. 3600 IN CNAME example.cdn-provider.net.
```

## [Content Security Policy (CSP)](https://daniele-michelin.com/en/glossary/content-security-policy/)

A Content Security Policy (CSP) is a response header that tells the browser from which sources a page may load scripts, stylesheets, images and frames.

```
Content-Security-Policy: default-src 'self'; img-src 'self' data:; frame-ancestors 'none'
```

## [Cron / cron job](https://daniele-michelin.com/en/glossary/cron/)

Cron is the classic Linux service that runs commands at scheduled times; a cron job is a single entry in a crontab.

```
*/5 * * * * php -f /var/www/nextcloud/cron.php
```

## [DKIM](https://daniele-michelin.com/en/glossary/dkim/)

DKIM (DomainKeys Identified Mail) signs outgoing email with a private key; receivers verify the signature with the public key published in DNS.

```
dig +short TXT mail2024._domainkey.example.com
```

## [DMARC](https://daniele-michelin.com/en/glossary/dmarc/)

DMARC is a DNS policy that tells receivers how to handle email whose From domain is not confirmed by SPF or DKIM.

```
_dmarc.example.com. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"
```

## [DNS propagation](https://daniele-michelin.com/en/glossary/dns-propagation/)

DNS propagation is the time it takes for a DNS change to reach all resolvers, because old answers first have to expire from their caches.

```
dig @ns1.example-dns.net example.com A +short; dig @8.8.8.8 example.com A +short
```

## [DNS resolver](https://daniele-michelin.com/en/glossary/dns-resolver/)

A recursive DNS resolver takes queries from clients, fetches the answer from the authoritative name servers and caches it.

```
dig @1.1.1.1 example.com A
```

## [DNSSEC](https://daniele-michelin.com/en/glossary/dnssec/)

DNSSEC adds cryptographic signatures to DNS so resolvers can verify that answers are authentic and unmodified.

```
dig +dnssec example.com SOA
```

## [Escalation](https://daniele-michelin.com/en/glossary/escalation/)

In IT support, an escalation is passing a ticket to someone with more expertise or more authority because it cannot be solved at the current level.

## [File descriptor](https://daniele-michelin.com/en/glossary/file-descriptor/)

A file descriptor is an integer a Linux process uses to access an open file, socket or pipe.

```
ls /proc/$(pgrep -o nginx)/fd | wc -l
```

## [Glue record](https://daniele-michelin.com/en/glossary/glue-record/)

A glue record is an A or AAAA record for a name server that the parent zone serves along with the delegation, so it can be resolved.

```
dig +norec @a.gtld-servers.net example.com NS
```

## [Greylisting](https://daniele-michelin.com/en/glossary/greylisting/)

Greylisting is a spam defence where the mail server temporarily rejects messages from unknown senders and only accepts them when delivery is retried.

```
451 4.7.1 <user@example.com>: Recipient address rejected: Greylisted, try again later
```

## [HSTS](https://daniele-michelin.com/en/glossary/hsts/)

HSTS (HTTP Strict Transport Security) is a response header that tells browsers to access a site exclusively over HTTPS for a set period of time.

```
Strict-Transport-Security: max-age=15552000; includeSubDomains
```

## [HTTP status code](https://daniele-michelin.com/en/glossary/http-status-code/)

An HTTP status code is the three-digit number a web server returns to report the result of a request.

```
curl -sI https://example.com/ | head -n 1
```

## [Identity provider (IdP)](https://daniele-michelin.com/en/glossary/identity-provider/)

An identity provider (IdP) is the central service that authenticates users and confirms their identity and attributes to applications.

## [IMAP](https://daniele-michelin.com/en/glossary/imap/)

IMAP (Internet Message Access Protocol) is the protocol mail clients use to read, move and manage messages directly on the server.

```
openssl s_client -connect mail.example.com:993 -quiet
```

## [Inode](https://daniele-michelin.com/en/glossary/inode/)

An inode is the data structure in a Linux file system that stores a file's metadata (permissions, owner, timestamps, location of data blocks), but not its name.

```
df -i /var
```

## [journald](https://daniele-michelin.com/en/glossary/journald/)

journald (systemd-journald) is the systemd logging service that collects messages from the kernel, services and applications in one structured journal.

```
journalctl -u php8.3-fpm --since "1 hour ago" -p warning
```

## [JWT (JSON Web Token)](https://daniele-michelin.com/en/glossary/json-web-token/)

A JWT (JSON Web Token) is a compact signed token that carries statements (claims) about a user or client in JSON form.

```
cut -d. -f2 token.txt | base64 -d 2>/dev/null
```

## [logrotate](https://daniele-michelin.com/en/glossary/logrotate/)

logrotate is a Linux tool that renames, compresses and deletes log files by time or size so the disk does not fill up.

```
logrotate -d /etc/logrotate.d/nginx
```

## [MTA (mail transfer agent)](https://daniele-michelin.com/en/glossary/mta-mail-transfer-agent/)

An MTA (mail transfer agent) is the server software that accepts email over SMTP, queues it and passes it to the next server or the mailbox.

```
postqueue -p | tail -n 5
```

## [MTA-STS](https://daniele-michelin.com/en/glossary/mta-sts/)

MTA-STS is a standard that lets a domain publish that inbound mail must only be delivered to its MX servers over TLS with a valid certificate.

```
curl -s https://mta-sts.example.com/.well-known/mta-sts.txt
```

## [Multi-factor authentication (MFA)](https://daniele-michelin.com/en/glossary/multi-factor-authentication/)

Multi-factor authentication (MFA) requires at least two independent proofs at login, from the categories knowledge, possession and biometrics.

## [MX record](https://daniele-michelin.com/en/glossary/mx-record/)

An MX record lists the mail servers that accept email for a domain, each with a priority.

```
dig +short MX example.com
```

## [Nextcloud](https://daniele-michelin.com/en/glossary/nextcloud/)

Nextcloud is an open-source platform for file sync, sharing, calendars, contacts and collaboration that you run on your own servers.

```
sudo -u www-data php occ status
```

## [NS record](https://daniele-michelin.com/en/glossary/ns-record/)

An NS record names the authoritative name servers responsible for a DNS zone.

```
dig +short NS example.com
```

## [NXDOMAIN](https://daniele-michelin.com/en/glossary/nxdomain/)

NXDOMAIN is the DNS response saying that the queried name does not exist, for any record type.

```
dig new.example.com | grep status
```

## [OAuth 2.0](https://daniele-michelin.com/en/glossary/oauth-2-0/)

OAuth 2.0 is an authorization framework that lets an application access a resource on behalf of a user without receiving the user's password.

## [OOM killer](https://daniele-michelin.com/en/glossary/oom-killer/)

The OOM (out of memory) killer is the Linux kernel function that forcibly terminates a process when memory is exhausted, to keep the system alive.

```
journalctl -k | grep -i 'out of memory'
```

## [OpenID Connect (OIDC)](https://daniele-michelin.com/en/glossary/openid-connect/)

OpenID Connect (OIDC) is a login protocol built on OAuth 2.0 that, in addition to access, delivers the user's identity as a signed ID token.

```
curl -s https://id.example.com/realms/main/.well-known/openid-configuration
```

## [PHP-FPM](https://daniele-michelin.com/en/glossary/php-fpm/)

PHP-FPM (FastCGI Process Manager) is a service that manages a pool of PHP processes and handles requests from the web server over FastCGI.

```
grep -E '^pm' /etc/php/8.3/fpm/pool.d/www.conf
```

## [PTR record](https://daniele-michelin.com/en/glossary/ptr-record/)

A PTR record provides reverse DNS: it maps an IP address back to a hostname.

```
dig +short -x 203.0.113.10
```

## [Relay / open relay](https://daniele-michelin.com/en/glossary/relay-open-relay/)

Relaying means a mail server accepts a message that is not addressed to its own domains and passes it on to another server.

```
postconf mynetworks smtpd_relay_restrictions
```

## [Reverse proxy](https://daniele-michelin.com/en/glossary/reverse-proxy/)

A reverse proxy is a server that accepts client requests and forwards them to one or more backend servers (upstreams).

```
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## [Root cause analysis (RCA)](https://daniele-michelin.com/en/glossary/root-cause-analysis/)

A root cause analysis (RCA) is the structured investigation of an incident aimed at finding the underlying cause and preventing it from happening again.

## [RPO and RTO](https://daniele-michelin.com/en/glossary/rpo-rto/)

RPO (recovery point objective) is the maximum acceptable data loss measured in time; RTO (recovery time objective) is the maximum acceptable downtime until recovery.

## [S3 object storage and bucket](https://daniele-michelin.com/en/glossary/s3-object-storage/)

S3 is the HTTP API for object storage introduced by Amazon and now supported by many providers; a bucket is the container that holds the objects.

```
aws s3 ls s3://backup-bucket --endpoint-url https://s3.example.com
```

## [SAML 2.0](https://daniele-michelin.com/en/glossary/saml/)

SAML 2.0 (Security Assertion Markup Language) is an XML-based standard in which an identity provider tells an application, with a signature, who has logged in.

```
https://cloud.example.com/apps/user_saml/saml/metadata
```

## [SAN (Subject Alternative Name)](https://daniele-michelin.com/en/glossary/subject-alternative-name/)

The SAN (Subject Alternative Name) is the TLS certificate field that lists all hostnames, and optionally IP addresses, the certificate is valid for.

```
openssl x509 -in fullchain.pem -noout -ext subjectAltName
```

## [SCIM](https://daniele-michelin.com/en/glossary/scim/)

SCIM (System for Cross-domain Identity Management) is a REST standard that lets an identity provider create, update and deactivate users and groups in applications.

## [Service provider (SP)](https://daniele-michelin.com/en/glossary/service-provider/)

In SSO, a service provider (SP) is the application that does not authenticate users itself but trusts an identity provider to do so.

## [Single sign-on (SSO)](https://daniele-michelin.com/en/glossary/single-sign-on/)

Single sign-on (SSO) means users authenticate once at a central service and can then use several applications without entering a password again.

## [SLA (service level agreement)](https://daniele-michelin.com/en/glossary/service-level-agreement/)

An SLA (service level agreement) is an agreement between a provider and a customer on measurable targets such as availability, response time and resolution time.

## [SMTP](https://daniele-michelin.com/en/glossary/smtp/)

SMTP (Simple Mail Transfer Protocol) is the protocol used to transfer email between mail servers and from mail clients to their server.

```
openssl s_client -connect mail.example.com:587 -starttls smtp
```

## [Snapshot vs backup](https://daniele-michelin.com/en/glossary/snapshot-vs-backup/)

A snapshot is a point-in-time image of a volume or VM on the same system; a backup is an independent copy of the data stored somewhere else.

```
lvcreate -s -n data_pre_update -L 10G /dev/vg0/data
```

## [SNI (Server Name Indication)](https://daniele-michelin.com/en/glossary/server-name-indication/)

SNI (Server Name Indication) is a TLS extension where the client names the requested hostname during the handshake, so the server can pick the right certificate.

```
openssl s_client -connect 203.0.113.10:443 -servername cloud.example.com
```

## [SOA record](https://daniele-michelin.com/en/glossary/soa-record/)

The SOA (Start of Authority) record holds a zone's administrative data, including the serial number and timers for secondary name servers.

```
dig +short SOA example.com
```

## [SPF](https://daniele-michelin.com/en/glossary/spf/)

SPF (Sender Policy Framework) is a TXT record that defines which servers are allowed to send email on behalf of a domain.

```
v=spf1 mx include:spf.protection.outlook.com -all
```

## [SRV record](https://daniele-michelin.com/en/glossary/srv-record/)

An SRV record tells clients on which host and port a specific service for a domain is available.

```
dig +short SRV _caldavs._tcp.example.com
```

## [STARTTLS](https://daniele-michelin.com/en/glossary/starttls/)

STARTTLS is a command that upgrades an initially unencrypted connection (SMTP, IMAP, POP3) to TLS.

```
openssl s_client -connect mail.example.com:25 -starttls smtp -servername mail.example.com
```

## [Submission (port 587)](https://daniele-michelin.com/en/glossary/submission-port-587/)

Submission is the SMTP service on port 587 where mail clients and applications authenticate and hand over email to their server for sending.

```
swaks --to test@example.org --server mail.example.com:587 --tls --auth-user user@example.com
```

## [systemd unit](https://daniele-michelin.com/en/glossary/systemd-unit/)

A systemd unit is a configuration file that systemd uses to manage a service, timer, mount, socket or other system object.

```
systemctl cat nginx.service
```

## [TLS](https://daniele-michelin.com/en/glossary/tls/)

TLS (Transport Layer Security) is the protocol that encrypts connections and proves the server's identity with certificates; it replaced SSL.

```
openssl s_client -connect example.com:443 -servername example.com -brief
```

## [TTL (time to live)](https://daniele-michelin.com/en/glossary/ttl/)

TTL (time to live) is the number of seconds a resolver may cache a DNS record.

```
dig example.com A +noall +answer
```

## [TXT record](https://daniele-michelin.com/en/glossary/txt-record/)

A TXT record stores arbitrary text for a DNS name and is used mostly for email authentication and ownership checks.

```
dig +short TXT example.com
```

## [WebDAV](https://daniele-michelin.com/en/glossary/webdav/)

WebDAV is an extension of HTTP that lets clients create, read, move and lock files and folders on a web server.

```
curl -u user -X PROPFIND -H 'Depth: 1' https://cloud.example.com/remote.php/dav/files/user/
```

---
More detail, related guides and tools: **[daniele-michelin.com/en/glossary/](https://daniele-michelin.com/en/glossary/)**
