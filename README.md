# sysadmin-notes

Practical notes from running mail servers, Nextcloud, DNS and Linux hosts. Short, copy-paste friendly, and tested in real operation.

- **116 error messages** with causes, checks and fixes (SMTP, DNS, Nextcloud, nginx, PHP, TLS …)
- **8 cheat sheets** with copyable commands
- **31 server ports** and what runs on them
- **64 glossary terms**

Everything here is generated from the data behind [daniele-michelin.com](https://daniele-michelin.com/en/), where each entry has a longer page, related guides and free tools (SPF lookup counter, DMARC generator, TLS and header checks, error explainer). German and Italian versions live there too.

## Cheat sheets

- [Nextcloud occ cheat sheet](cheat-sheets/nextcloud-occ-cheat-sheet.md) (35 commands)
- [dig cheat sheet](cheat-sheets/dig-cheat-sheet.md) (30 commands)
- [journalctl cheat sheet](cheat-sheets/journalctl-cheat-sheet.md) (31 commands)
- [OpenSSL cheat sheet](cheat-sheets/openssl-cheat-sheet.md) (33 commands)
- [nginx cheat sheet](cheat-sheets/nginx-cheat-sheet.md) (37 commands)
- [systemctl cheat sheet](cheat-sheets/systemctl-cheat-sheet.md) (37 commands)
- [Docker cheat sheet](cheat-sheets/docker-cheat-sheet.md) (38 commands)
- [Git cheat sheet](cheat-sheets/git-cheat-sheet.md) (37 commands)

## Reference

- [Server ports](ports.md)
- [Glossary](glossary.md)

## Error messages

| Code | Meaning |
|---|---|
| [ERROR 1040 (08004): Too many connections](errors/mysql-error-1040-too-many-connections.md) | The server has reached `max_connections` and rejects every new client |
| [ERROR 1045 (28000)](errors/mysql-1045-access-denied.md) | The server was reached, but it rejects the login: wrong password, a user that does not exist for this host, or an authentication plugin that does not allow a password |
| [ERROR 2002 (HY000)](errors/mysql-error-2002-cant-connect-through-socket.md) | The client looks for the server's Unix socket file and does not find a running server there – usually the database is stopped, or client and server use different socket paths |
| [FATAL: no pg_hba.conf entry for host](errors/postgresql-fatal-no-pg-hba-conf-entry-for-host.md) | The server is reachable, but its access file `pg_hba.conf` has no line that fits this client: address, user, database and SSL mode |
| [FATAL: password authentication failed for user](errors/postgresql-fatal-password-authentication-failed.md) | The server found a matching `pg_hba.conf` line that asks for a password, and the password was not accepted |
| [MySQL server has gone away](errors/mysql-server-has-gone-away-2006.md) | The client sent a query on a connection that the server had already closed |
| [Table is marked as crashed and should be repaired](errors/mariadb-table-is-marked-as-crashed-and-should-be-repaired.md) | The table files of a MyISAM or Aria table are inconsistent, usually because the server stopped in the middle of a write |
| [CNAME at apex](errors/dns-cname-at-zone-apex-not-allowed.md) | A CNAME may not coexist with other records of the same name |
| [NXDOMAIN](errors/dns-nxdomain-dns-probe-finished-nxdomain.md) | The responsible name server answered authoritatively that this name does not exist |
| [SERVFAIL](errors/dns-servfail-dnssec.md) | The resolver could not produce a valid answer – either the authoritative nameservers are not answering correctly or DNSSEC validation fails |
| [connection timed out; no servers could be reached](errors/dig-connection-timed-out-no-servers-could-be-reached.md) | The query got no answer at all, not even a refusal |
| [curl: (6) Could not resolve host](errors/curl-6-could-not-resolve-host.md) | curl could not turn the host name into an IP address, so it never tried to connect |
| [status: REFUSED](errors/dns-status-refused-query-refused.md) | The server was reached and answered on purpose with “no” |
| [400](errors/http-400-request-header-or-cookie-too-large.md) | A single request header or the cookie line is larger than the buffer nginx reserves for it |
| [403](errors/http-403-forbidden-nginx.md) | nginx understands the request but refuses access |
| [404 Not Found](errors/http-404-not-found-nginx.md) | nginx looked for a file at the path it built from `root` or `alias` and the request URI, and did not find it |
| [413](errors/http-413-request-entity-too-large.md) | The request body (usually an upload) is larger than the limit nginx or a proxy in front of it allows – the request never reaches the application |
| [429 Too Many Requests](errors/http-429-too-many-requests.md) | The server or an intermediate layer says that this client sent too many requests in a given time |
| [500](errors/http-500-internal-server-error-php.md) | The application or web server failed while handling the request |
| [502](errors/http-502-bad-gateway.md) | nginx could not get a valid response from the upstream (PHP-FPM, Node.js, Gunicorn, a Docker container) – the backend is down, unreachable or crashed |
| [503](errors/http-503-service-unavailable-nginx.md) | The service is temporarily unavailable |
| [504](errors/http-504-gateway-timeout.md) | nginx reached the backend but did not get a response within the configured timeout (60 seconds by default) |
| [ERR_TOO_MANY_REDIRECTS](errors/err-too-many-redirects-redirect-loop.md) | Two places keep sending the browser to each other |
| [Error 521 / 522 (origin unreachable behind CDN)](errors/http-521-522-origin-error-cdn.md) | These codes come from the CDN (for example Cloudflare), not from your server |
| [Mixed Content: requested an insecure resource](errors/mixed-content-blocked-insecure-resource.md) | The page itself uses HTTPS, but it tries to load a script, stylesheet, font, image or request over plain HTTP |
| [Argument list too long](errors/bash-argument-list-too-long.md) | The shell replaced `*` with every matching file name, and the list got larger than the kernel allows for one command |
| [Could not get lock /var/lib/dpkg/lock-frontend](errors/apt-could-not-get-lock-dpkg-lock-frontend.md) | Another process is using the package system right now, most often the automatic updater |
| [E: Unable to locate package](errors/apt-e-unable-to-locate-package.md) | apt does not find the package in any package list it knows |
| [EMFILE](errors/linux-too-many-open-files.md) | A process hit its file descriptor limit |
| [ENOSPC](errors/linux-no-space-left-on-device.md) | The kernel returns ENOSPC because a filesystem resource is exhausted – not necessarily disk space, often the inodes or space held by deleted files that are still open |
| [EROFS](errors/linux-read-only-file-system.md) | A write was refused because the filesystem is read-only |
| [Out of memory](errors/linux-out-of-memory-killed-process-oom-killer.md) | The system or a cgroup ran out of memory |
| [Permission denied (publickey)](errors/ssh-permission-denied-publickey.md) | The server only allows key login and accepted none of the keys offered |
| [REMOTE HOST IDENTIFICATION HAS CHANGED](errors/ssh-remote-host-identification-has-changed.md) | The server presents a different host key than the one stored in your `known_hosts` |
| [Temporary failure in name resolution](errors/temporary-failure-in-name-resolution.md) | The program asked the system resolver for an IP address and got no answer at all |
| [Too many authentication failures](errors/ssh-too-many-authentication-failures.md) | The server counts every key your SSH agent offers as a failed attempt |
| [control process exited](errors/systemd-job-failed-control-process-exited-with-error-code.md) | systemd tried to start or reload the service and the start command (ExecStart or ExecStartPre) exited with an error |
| [dpkg was interrupted, you must manually run 'dpkg --configure -a'](errors/dpkg-was-interrupted-configure-a.md) | A previous install or upgrade did not finish |
| [is not in the sudoers file. This incident will be reported.](errors/sudo-user-is-not-in-the-sudoers-file.md) | sudo works, but the policy does not allow this user |
| [permission denied docker.sock](errors/docker-permission-denied-docker-sock.md) | Your user may not use `/var/run/docker.sock` |
| [port 22: Connection timed out](errors/ssh-connect-to-host-port-22-connection-timed-out.md) | Your packets get no answer at all |
| [start-limit-hit](errors/systemd-start-request-repeated-too-quickly.md) | The service crashed again and again in a short time, so systemd stopped restarting it to protect the system |
| [wrong fs type, bad option, bad superblock](errors/mount-wrong-fs-type-bad-option-bad-superblock.md) | mount is a catch-all message |
| [423 Locked](errors/nextcloud-file-is-locked-423.md) | Nextcloud refuses access because a lock still exists for the file in transactional file locking – usually a stale lock after an aborted upload or crash |
| [APCu not available](errors/nextcloud-memcache-apcu-not-available-for-local-cache.md) | Nextcloud is configured to use APCu as local cache, but the PHP instance currently running has APCu disabled |
| [CSRF check failed](errors/nextcloud-csrf-check-failed.md) | The CSRF token in the form does not match the session on the server |
| [Cannot write into "config" directory](errors/nextcloud-cannot-write-into-config-directory.md) | The web server user may not write to `config/`, so the installer, updates or settings cannot save `config.php` |
| [Internal Server Error](errors/nextcloud-internal-server-error-request-id.md) | Nextcloud hit an error while handling the request |
| [Last background job](errors/nextcloud-last-background-job-ran-ago-cron.md) | Nextcloud handles cleanup, notifications, file versions and trash via background jobs |
| [RedisException: Redis went away](errors/nextcloud-redisexception-redis-went-away.md) | Nextcloud uses Redis for file locking and caching |
| [Some files have not passed the integrity check](errors/nextcloud-some-files-have-not-passed-the-integrity-check.md) | Nextcloud compares its code files with a signed list of hashes |
| [Too many requests](errors/nextcloud-too-many-requests-brute-force-protection.md) | Nextcloud's brute-force protection has slowed down or blocked your IP address after repeated failed logins |
| [Updates between multiple major versions and downgrades are unsupported](errors/nextcloud-updates-between-multiple-major-versions-unsupported.md) | Nextcloud only upgrades from one major version to the next |
| [Your data directory is invalid](errors/nextcloud-your-data-directory-is-invalid-ocdata.md) | Nextcloud refuses to start because it does not find the marker file `.ocdata` in the data directory |
| [Your web server is not properly set up to resolve "/.well-known/caldav"](errors/nextcloud-well-known-caldav-carddav-not-set-up.md) | Calendar and contact apps find your server through the `/.well-known/caldav` and `/.well-known/carddav` addresses |
| [data directory readable](errors/nextcloud-data-directory-readable-by-other-users.md) | On start Nextcloud checks whether other system users can read or list the data directory |
| [maintenance mode](errors/nextcloud-maintenance-mode-enabled.md) | Nextcloud is in maintenance mode and blocks the web UI, clients and most occ commands – either set on purpose or left behind by an update that did not finish cleanly |
| [missing indexes](errors/nextcloud-database-is-missing-some-indexes.md) | After an update Nextcloud defined new indexes but did not create them automatically, because that can take long on large tables |
| [reverse proxy header configuration is incorrect](errors/nextcloud-reverse-proxy-header-configuration-incorrect.md) | Nextcloud sees forwarded headers (X-Forwarded-For) from a proxy it does not trust, or no proxy is configured although one is in front |
| [untrusted domain](errors/nextcloud-trusted-domain-error.md) | Nextcloud was opened via a hostname or IP that is not listed in `trusted_domains`, so it refuses to log you in |
| [110: Connection timed out](errors/nginx-upstream-timed-out-110.md) | The backend (PHP-FPM, Node, Gunicorn) did not answer within nginx's time limit, so nginx gives up and returns 504 Gateway Timeout |
| [111: Connection refused](errors/nginx-connect-failed-111-connection-refused-upstream.md) | nginx reaches the backend host, but nothing listens on the target port |
| [98: Address already in use](errors/nginx-bind-address-already-in-use.md) | nginx cannot open its port because another process already listens on it – often Apache, a second nginx or a Docker container |
| [Primary script unknown](errors/nginx-php-fpm-primary-script-unknown-file-not-found.md) | nginx passes the request to PHP-FPM, but PHP-FPM finds no file at the given path or may not read it |
| [cannot load certificate](errors/nginx-emerg-cannot-load-certificate-pem-no-start-line.md) | nginx could not read the certificate or the key it was told to use, so it refuses to start or reload |
| [connect() to unix:/run/php/php-fpm.sock failed (13: Permission denied)](errors/nginx-php-fpm-socket-13-permission-denied.md) | nginx found the PHP-FPM socket but is not allowed to open it |
| [could not build server_names_hash](errors/nginx-could-not-build-server-names-hash.md) | nginx stores all `server_name` values in a hash table |
| [rewrite or internal redirection cycle](errors/nginx-rewrite-or-internal-redirection-cycle.md) | nginx redirected the request internally again and again and gave up after 10 steps, answering 500 |
| [too big header](errors/nginx-upstream-sent-too-big-header.md) | The backend's response headers (usually cookies) are larger than the buffer nginx reserves for them; nginx gives up and returns 502 Bad Gateway |
| [upstream prematurely closed connection](errors/nginx-upstream-prematurely-closed-connection.md) | nginx sent the request to the backend, but the backend closed the connection before it sent a response |
| [worker_connections are not enough](errors/nginx-worker-connections-are-not-enough.md) | One nginx worker has used up all connections it may hold at once |
| [Allowed memory size exhausted](errors/php-allowed-memory-size-exhausted.md) | A PHP script used more memory than `memory_limit` allows, so PHP stopped it |
| [Call to undefined function mysqli_connect()](errors/php-call-to-undefined-function-mysqli-connect.md) | The PHP that runs your page has no `mysqli` extension loaded |
| [Cannot modify header information - headers already sent](errors/php-cannot-modify-header-information-headers-already-sent.md) | PHP sends HTTP headers (redirect, cookie, session) only before any body output |
| [Maximum execution time exceeded](errors/php-maximum-execution-time-exceeded.md) | A PHP script ran longer than `max_execution_time` (30 seconds by default for web requests) and PHP stopped it |
| [The uploaded file exceeds the upload_max_filesize directive in php.ini](errors/php-uploaded-file-exceeds-upload-max-filesize.md) | PHP rejected the upload because the file is larger than `upload_max_filesize`, or the whole request is larger than `post_max_size` |
| [server reached pm.max_children setting](errors/php-fpm-server-reached-pm-max-children.md) | All PHP workers of the pool are busy |
| [421 4.7.0](errors/smtp-421-4-7-0.md) | The receiving server temporarily refuses more mail from your IP because volume, reputation or authentication look suspicious – messages stay in the queue and are retried later |
| [450 4.1.8 Sender address rejected: Domain not found](errors/smtp-450-4-1-8-sender-address-rejected-domain-not-found.md) | The receiving server looked up the domain in the sender address and found neither an A nor an MX record, so it refuses the mail with `reject_unknown_sender_domain` |
| [451 4.7.1](errors/smtp-451-4-7-1-greylisting.md) | The receiver deliberately rejects the first delivery attempt from an unknown sender; a properly working mail server retries after a few minutes and gets through |
| [530 5.7.0](errors/smtp-530-5-7-0-must-issue-starttls.md) | On this port the server accepts neither AUTH nor MAIL FROM over an unencrypted connection |
| [535 5.7.8](errors/smtp-535-5-7-8-authentication-failed.md) | The server rejected the SMTP AUTH login |
| [550 5.1.1](errors/smtp-550-5-1-1-user-unknown.md) | The target server does not know the recipient address and rejects the message permanently |
| [550 5.4.1 Recipient address rejected: Access denied](errors/smtp-550-5-4-1-recipient-address-rejected-access-denied.md) | This bounce comes from Exchange Online when it protects itself against address harvesting |
| [550 5.7.1](errors/smtp-550-5-7-1.md) | The server rejects the message for policy reasons – usually because the sender is not authenticated and the server would have to relay mail to a domain it is not responsible for |
| [550 5.7.26](errors/smtp-550-5-7-26.md) | Gmail (and Microsoft with 5.7.509/5.7.515) rejects the message because it is not authenticated via SPF or DKIM in alignment with the From domain |
| [552 5.2.2](errors/smtp-552-5-2-2-mailbox-full.md) | The receiving server refuses the message because the recipient's mailbox has hit its quota |
| [553 5.7.1](errors/smtp-553-5-7-1-sender-address-rejected.md) | Postfix compares the sender address (MAIL FROM) with the authenticated SASL user |
| [554 5.7.1](errors/smtp-554-5-7-1.md) | The receiver permanently rejects the connection or message because the sending IP is on a DNS blocklist (DNSBL) or the content was classified as spam |
| [554 5.7.1 Relay access denied](errors/smtp-554-5-7-1-relay-access-denied.md) | The server is asked to deliver mail to a domain it does not own, from a client it does not trust |
| [Dovecot Permission denied](errors/dovecot-mailbox-open-failed-permission-denied.md) | The user Dovecot runs as for this mailbox may not read or write the mail files |
| [Host or domain name not found](errors/postfix-host-or-domain-name-not-found-name-service-error.md) | Postfix could not look up the recipient domain in DNS |
| [SASL authentication failed](errors/postfix-sasl-authentication-failed-no-mechanism-available.md) | Postfix tried to log in at your smart host (relay) and failed |
| [auth failed](errors/dovecot-imap-login-disconnected-auth-failed.md) | A client logged in to IMAP or POP3 and was refused |
| [dkim=fail body hash did not verify](errors/dkim-fail-body-hash-did-not-verify.md) | The signature is intact, but the body of the message no longer matches the hash that was signed |
| [does not resolve to address](errors/postfix-hostname-does-not-resolve-to-address.md) | Postfix found a name for the connecting server's IP via PTR, but that name does not point back to the same IP |
| [mail for example.com loops back to myself](errors/postfix-mail-for-domain-loops-back-to-myself.md) | Postfix looked up where to send the mail and ended up at itself, so it stops instead of looping forever |
| [not accepted due to domain's DMARC policy](errors/dmarc-unauthenticated-email-not-accepted-dmarc-policy.md) | The recipient checked your domain's DMARC policy, and the message passed neither SPF nor DKIM in a way that matches the From domain, so it was rejected as the policy says (`p=reject` or `quarantine`) |
| [status=deferred Connection timed out](errors/postfix-deferred-connect-connection-timed-out.md) | Your mail server cannot open a connection to the recipient's server on port 25, so it keeps the message in the queue and tries again later |
| [Invalid audience for this Response](errors/saml-invalid-audience-for-this-response.md) | The identity provider addresses its assertion to one audience, the Entity ID of the service provider |
| [NotBefore](errors/saml-clock-skew-notbefore-timestamp-not-yet-valid.md) | Every SAML assertion is only valid within a time window (NotBefore to NotOnOrAfter) |
| [SAML signature](errors/nextcloud-saml-signature-validation-failed.md) | Nextcloud (user_saml with php-saml) could not verify the SAML response signature with the stored IdP certificate and rejects the login |
| [invalid_client](errors/oidc-invalid-client-client-authentication-failed.md) | After the user signs in, your app calls the token endpoint and has to prove who it is with its client ID and secret |
| [invalid_grant](errors/oidc-invalid-grant-token-endpoint.md) | The identity provider rejects the authorization code or refresh token the client wants to exchange |
| [redirect_uri_mismatch](errors/oidc-redirect-uri-mismatch-invalid-redirect-uri.md) | In the authorize request the client sends a redirect_uri that is not registered exactly like that at the identity provider |
| [ERR_CERT_COMMON_NAME_INVALID](errors/net-err-cert-common-name-invalid.md) | The certificate is valid but not issued for this name |
| [ERR_SSL_VERSION_OR_CIPHER_MISMATCH](errors/tls-err-ssl-version-or-cipher-mismatch.md) | The browser and the server could not agree on a TLS version or cipher |
| [Failed to renew certificate](errors/certbot-failed-to-renew-certificate-renewals-failed.md) | Let's Encrypt certificates last a short time, so renewal must work on its own |
| [NET::ERR_CERT_AUTHORITY_INVALID](errors/net-err-cert-authority-invalid-self-signed.md) | The browser got a certificate but cannot trace it back to a certificate authority it trusts |
| [NET::ERR_CERT_DATE_INVALID](errors/net-err-cert-date-invalid-clock-wrong.md) | A certificate is only valid between two dates |
| [SSL_ERROR_RX_RECORD_TOO_LONG](errors/ssl-error-rx-record-too-long.md) | The browser expects a TLS handshake on port 443 but receives plain text, usually an HTTP response |
| [Timeout during connect (likely firewall problem)](errors/certbot-timeout-during-connect-firewall.md) | Let's Encrypt could not open a connection to your server on port 80 to fetch the challenge file |
| [acme:error:unauthorized](errors/letsencrypt-acme-error-unauthorized.md) | Let's Encrypt could not fetch the challenge file under `/.well-known/acme-challenge/` from the right server and therefore refuses to issue the certificate |
| [certificate has expired](errors/ssl-certificate-has-expired.md) | The served certificate is outside its validity period |
| [rateLimited](errors/letsencrypt-too-many-certificates-already-issued.md) | Let's Encrypt limits how many certificates are issued in a time window |
| [unable to get local issuer certificate](errors/ssl-unable-to-get-local-issuer-certificate.md) | The client cannot build the chain from the server certificate to a trusted root CA |

## Free tools

- [SPF lookup limit checker](https://daniele-michelin.com/en/tools/spf-lookup-checker/)
- [Certificate check](https://daniele-michelin.com/en/tools/ssl-certificate-checker/)
- [Error explainer](https://daniele-michelin.com/en/tools/explain-error-message/)
- [Mail DNS check](https://daniele-michelin.com/en/tools/mail-dns-checker/)

Badges for your own README (live, refreshed every 6 hours):

```markdown
![SPF](https://daniele-michelin.com/tools/api/badge.php?d=example.com)
![TLS](https://daniele-michelin.com/tools/api/badge.php?t=tls&d=example.com)
![headers](https://daniele-michelin.com/tools/api/badge.php?t=headers&d=example.com)
```

## License

Text under [CC BY 4.0](LICENSE): share and adapt it, just credit [Daniele Michelin](https://daniele-michelin.com/en/about/) and link back.
