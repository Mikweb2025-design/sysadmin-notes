# Common server ports

| Port | Protocol | Service | Notes |
|---|---|---|---|
| 20-21 | TCP | [FTP](https://daniele-michelin.com/en/ports/20-21/) | FTP uses port 21 for commands and port 20 (active mode) or a passive port range for data transfers. |
| 22 | TCP | [SSH / SFTP](https://daniele-michelin.com/en/ports/22/) | Port 22 is the standard port for SSH remote access, SFTP and scp. |
| 25 | TCP | [SMTP (MTA-to-MTA)](https://daniele-michelin.com/en/ports/25/) | Port 25 carries email between mail servers; clients should use 587 or 465 instead. |
| 53 | TCP/UDP | [DNS](https://daniele-michelin.com/en/ports/53/) | Port 53 is the DNS port – UDP for normal queries, TCP for large responses and zone transfers. |
| 80 | TCP | [HTTP](https://daniele-michelin.com/en/ports/80/) | Port 80 is the unencrypted HTTP port, today mainly used for redirects and ACME challenges. |
| 110 | TCP | [POP3](https://daniele-michelin.com/en/ports/110/) | Port 110 is the POP3 port for downloading mail, encrypted only via STARTTLS. |
| 123 | UDP | [NTP](https://daniele-michelin.com/en/ports/123/) | Port 123/UDP is used by NTP to keep system clocks synchronized. |
| 143 | TCP | [IMAP](https://daniele-michelin.com/en/ports/143/) | Port 143 is the IMAP port, encrypted via STARTTLS after the connection is established. |
| 389 | TCP/UDP | [LDAP](https://daniele-michelin.com/en/ports/389/) | Port 389 is the LDAP port for directory services such as OpenLDAP and Active Directory. |
| 443 | TCP/UDP | [HTTPS / HTTP/3](https://daniele-michelin.com/en/ports/443/) | Port 443 is the HTTPS port – TCP for HTTP/1.1 and HTTP/2, UDP for HTTP/3. |
| 465 | TCP | [SMTP Submission (implicit TLS)](https://daniele-michelin.com/en/ports/465/) | Port 465 is the submission port with implicit TLS for authenticated mail sending. |
| 587 | TCP | [SMTP Submission](https://daniele-michelin.com/en/ports/587/) | Port 587 is the standard port for authenticated mail submission from clients and apps using STARTTLS. |
| 636 | TCP | [LDAPS](https://daniele-michelin.com/en/ports/636/) | Port 636 is the LDAPS port: LDAP with TLS from the first byte. |
| 853 | TCP/UDP | [DNS over TLS / DNS over QUIC](https://daniele-michelin.com/en/ports/853/) | Port 853 carries encrypted DNS: TCP for DNS over TLS, UDP for DNS over QUIC. |
| 993 | TCP | [IMAPS](https://daniele-michelin.com/en/ports/993/) | Port 993 is the IMAP port with implicit TLS and the default choice for mail clients. |
| 995 | TCP | [POP3S](https://daniele-michelin.com/en/ports/995/) | Port 995 is the POP3 port with implicit TLS. |
| 1194 | UDP | [OpenVPN](https://daniele-michelin.com/en/ports/1194/) | Port 1194 is the default OpenVPN port, normally over UDP. |
| 2049 | TCP/UDP | [NFS](https://daniele-michelin.com/en/ports/2049/) | Port 2049 is the NFS server port for network file shares on Linux. |
| 3306 | TCP | [MySQL / MariaDB](https://daniele-michelin.com/en/ports/3306/) | Port 3306 is the default port of MySQL and MariaDB. |
| 3389 | TCP/UDP | [RDP](https://daniele-michelin.com/en/ports/3389/) | Port 3389 is the port of the Windows Remote Desktop Protocol (RDP). |
| 3478 | TCP/UDP | [STUN / TURN (coturn, Nextcloud Talk)](https://daniele-michelin.com/en/ports/3478/) | Port 3478 is the default STUN and TURN port, e.g. coturn for Nextcloud Talk. |
| 4190 | TCP | [ManageSieve](https://daniele-michelin.com/en/ports/4190/) | Port 4190 is the ManageSieve port for managing server-side Sieve mail filters. |
| 5432 | TCP | [PostgreSQL](https://daniele-michelin.com/en/ports/5432/) | Port 5432 is the default port of PostgreSQL. |
| 6379 | TCP | [Redis](https://daniele-michelin.com/en/ports/6379/) | Port 6379 is the default port of Redis, an in-memory data store and cache. |
| 8080 | TCP | [HTTP alternate](https://daniele-michelin.com/en/ports/8080/) | Port 8080 is the most common alternate HTTP port for backends, proxies and containers. |
| 8443 | TCP | [HTTPS alternate / Plesk](https://daniele-michelin.com/en/ports/8443/) | Port 8443 is a widely used alternate HTTPS port, among others for the Plesk panel. |
| 9000 | TCP | [PHP-FPM (FastCGI)](https://daniele-michelin.com/en/ports/9000/) | Port 9000 is the usual TCP port of PHP-FPM for FastCGI requests from the web server. |
| 9090 | TCP | [Prometheus / Cockpit](https://daniele-michelin.com/en/ports/9090/) | Port 9090 is the default port of the Prometheus server and, on RHEL/Fedora, of the Cockpit web console. |
| 9100 | TCP | [Prometheus node_exporter](https://daniele-michelin.com/en/ports/9100/) | Port 9100 is the default port of the Prometheus node_exporter for Linux system metrics. |
| 11211 | TCP/UDP | [Memcached](https://daniele-michelin.com/en/ports/11211/) | Port 11211 is the default port of Memcached, a simple in-memory cache. |
| 51820 | UDP | [WireGuard](https://daniele-michelin.com/en/ports/51820/) | Port 51820/UDP is the conventional (not IANA-registered) default port of WireGuard. |
---
More detail, related guides and tools: **[daniele-michelin.com/en/ports/](https://daniele-michelin.com/en/ports/)**
