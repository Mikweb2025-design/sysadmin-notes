# OpenSSL cheat sheet

Check certificates, debug chains, convert formats – everyday openssl commands, searchable and copyable in one click.

## Test connections

- Connect with SNI and show the TLS handshake

  ```bash
  openssl s_client -connect example.com:443 -servername example.com </dev/null
  ```
- Show the full certificate chain served

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -showcerts </dev/null
  ```
- Validity period of the server certificate

  ```bash
  openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -dates
  ```
- Subject, issuer and SANs of the server cert

  ```bash
  openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -ext subjectAltName
  ```
- Check OCSP stapling

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -status </dev/null 2>/dev/null | grep -A 3 'OCSP Response'
  ```
- Test STARTTLS on SMTP submission

  ```bash
  openssl s_client -starttls smtp -connect mail.example.com:587 -servername mail.example.com </dev/null
  ```
- Test STARTTLS on IMAP

  ```bash
  openssl s_client -starttls imap -connect mail.example.com:143 </dev/null
  ```
- Test IMAPS (implicit TLS)

  ```bash
  openssl s_client -connect mail.example.com:993 -servername mail.example.com </dev/null
  ```

## TLS versions & ciphers

- Check whether TLS 1.3 is offered

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -tls1_3 </dev/null
  ```
- Check whether TLS 1.2 is offered

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -tls1_2 </dev/null
  ```
- Test legacy TLS 1.1 (should fail)

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -tls1_1 </dev/null
  ```
- Force a specific cipher suite

  ```bash
  openssl s_client -connect example.com:443 -servername example.com -tls1_2 -cipher 'ECDHE-RSA-AES256-GCM-SHA384' </dev/null
  ```
- List ciphers matched by a cipher string

  ```bash
  openssl ciphers -v 'HIGH:!aNULL:!MD5'
  ```

## Inspect certificates

- Show a certificate in full, human-readable

  ```bash
  openssl x509 -in cert.pem -noout -text
  ```
- Show the expiry date

  ```bash
  openssl x509 -in cert.pem -noout -enddate
  ```
- Expires within 30 days? (exit code 1 = yes)

  ```bash
  openssl x509 -in cert.pem -noout -checkend 2592000
  ```
- Show the SHA-256 fingerprint

  ```bash
  openssl x509 -in cert.pem -noout -fingerprint -sha256
  ```
- Verify the chain against root and intermediate

  ```bash
  openssl verify -CAfile root.pem -untrusted intermediate.pem cert.pem
  ```

## Keys & CSR

- Create key and CSR with SANs in one step

  ```bash
  openssl req -new -newkey rsa:4096 -nodes -keyout example.key -out example.csr -subj "/CN=example.com" -addext "subjectAltName=DNS:example.com,DNS:www.example.com"
  ```
- Generate an ECDSA key (P-256)

  ```bash
  openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out example.key
  ```
- Show a CSR and verify its signature

  ```bash
  openssl req -in example.csr -noout -text -verify
  ```
- Create a self-signed test certificate

  ```bash
  openssl req -x509 -newkey rsa:2048 -nodes -days 365 -keyout test.key -out test.crt -subj "/CN=test.local"
  ```
- Do key and cert match? Hashes must be equal

  ```bash
  openssl x509 -in cert.pem -noout -pubkey | openssl sha256; openssl pkey -in key.pem -pubout | openssl sha256
  ```
- RSA match via modulus (classic method)

  ```bash
  openssl x509 -in cert.pem -noout -modulus | openssl md5; openssl rsa -in key.pem -noout -modulus | openssl md5
  ```

## Convert formats

- Convert PEM to DER

  ```bash
  openssl x509 -in cert.pem -outform der -out cert.der
  ```
- Convert DER to PEM

  ```bash
  openssl x509 -in cert.der -inform der -out cert.pem
  ```
- Build a PFX/P12 from key, cert and chain (Windows/IIS)

  ```bash
  openssl pkcs12 -export -out cert.pfx -inkey key.pem -in cert.pem -certfile chain.pem
  ```
- Extract a PFX – the key ends up unencrypted (**careful**)

  ```bash
  openssl pkcs12 -in cert.pfx -nodes -out all.pem
  ```
- Remove the passphrase from a key (**careful**)

  ```bash
  openssl pkey -in encrypted.key -out plain.key
  ```

## Utilities

- Generate a random password (32 chars)

  ```bash
  openssl rand -base64 24
  ```
- Random secret as hex (e.g. for tokens)

  ```bash
  openssl rand -hex 32
  ```
- SHA-256 checksum of a file

  ```bash
  openssl dgst -sha256 file.iso
  ```
- Show OpenSSL version and build options

  ```bash
  openssl version -a
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/openssl-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/openssl-cheat-sheet/)**
