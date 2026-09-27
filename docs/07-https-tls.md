# Stage 07 — HTTPS, TLS and Internal PKI

## Overview

This stage added HTTPS/TLS to the Nginx web server running on VM4.

Instead of using a public certificate authority, the laboratory implemented its own internal Public Key Infrastructure (PKI).

The final HTTPS endpoint is:

```text
https://web.lab.test/
```

The server certificate was issued specifically for:

```text
web.lab.test
```

by an internal root certificate authority:

```text
LAB Root CA
```

HTTP on TCP port 80 was preserved during the implementation.

## Architecture

| Component | Role | Address |
|---|---|---|
| Windows | Administration and HTTPS testing | `192.168.56.1` |
| VM2 | DNS / BIND9 | `192.168.56.20` |
| VM3 | DHCP / Kea DHCP4 | `192.168.56.10` |
| VM4 | Nginx / HTTP / HTTPS | `192.168.56.30` |
| VM6 | Linux test client | `192.168.56.111` during testing |

Internal network:

```text
192.168.56.0/24
```

Internal domain:

```text
lab.test
```

DNS record:

```text
web.lab.test -> 192.168.56.30
```

## Initial Validation

Before changing the Nginx configuration, the existing environment was verified.

VM4 retained both network interfaces:

```text
enp0s3 -> NAT
enp0s8 -> 192.168.56.30/24
```

Nginx was confirmed as:

```text
active
enabled
```

HTTP remained functional at:

```text
http://web.lab.test/
```

Before HTTPS was configured, Nginx was listening only on TCP port 80.

TCP port 443 was available.

## Existing Nginx Site

The enabled Nginx site was:

```text
/etc/nginx/sites-enabled/default
```

which referenced:

```text
/etc/nginx/sites-available/default
```

The document root remained:

```text
/var/www/html
```

The HTTPS configuration was added to this existing site.

## Internal PKI Design

The PKI was separated into a root certificate authority and a dedicated server certificate.

```text
LAB Root CA
├── Private CA key
└── Public CA certificate

web.lab.test
├── Private server key
├── Certificate Signing Request
└── Server certificate signed by LAB Root CA
```

The following directories were created:

```text
/etc/ssl/lab-ca/private/
/etc/ssl/lab-ca/certs/

/etc/ssl/web.lab.test/private/
/etc/ssl/web.lab.test/csr/
/etc/ssl/web.lab.test/certs/
```

Private directories were restricted to:

```text
700
```

Private key files were restricted to:

```text
600
```

## Root Certificate Authority

A 4096-bit RSA private key was generated for the internal CA.

Private key:

```text
/etc/ssl/lab-ca/private/lab-ca.key
```

Public CA certificate:

```text
/etc/ssl/lab-ca/certs/lab-ca.crt
```

The root certificate was created with:

```text
Subject: C=BR, O=LAB, CN=LAB Root CA
Issuer:  C=BR, O=LAB, CN=LAB Root CA
```

Relevant X.509 properties included:

```text
Basic Constraints: CA:TRUE
Key Usage: Certificate Sign, CRL Sign
```

The root certificate was configured with a ten-year validity period.

The CA certificate is public and may be distributed to trusted laboratory clients.

The CA private key must remain private.

## Server Private Key

VM4 received its own dedicated RSA private key.

Key size:

```text
2048 bits
```

File:

```text
/etc/ssl/web.lab.test/private/web.lab.test.key
```

The key was validated before being used to create the certificate request.

## Certificate Signing Request

A Certificate Signing Request (CSR) was generated for the web server.

File:

```text
/etc/ssl/web.lab.test/csr/web.lab.test.csr
```

The request contained:

```text
Common Name: web.lab.test
Subject Alternative Name: DNS:web.lab.test
```

The Subject Alternative Name was explicitly included so that clients could validate the DNS hostname correctly.

## Server Certificate

The CSR was signed by the internal `LAB Root CA`.

Certificate file:

```text
/etc/ssl/web.lab.test/certs/web.lab.test.crt
```

Relevant certificate properties were:

```text
Subject: C=BR, O=LAB, CN=web.lab.test
Issuer: C=BR, O=LAB, CN=LAB Root CA

Basic Constraints: CA:FALSE
Key Usage: Digital Signature, Key Encipherment
Extended Key Usage: TLS Web Server Authentication
Subject Alternative Name: DNS:web.lab.test
```

The server certificate was issued with a one-year validity period.

## Certificate Chain Validation

The certificate chain was validated using OpenSSL.

```bash
openssl verify \
  -CAfile /etc/ssl/lab-ca/certs/lab-ca.crt \
  /etc/ssl/web.lab.test/certs/web.lab.test.crt
```

Result:

```text
/etc/ssl/web.lab.test/certs/web.lab.test.crt: OK
```

The server private key and certificate were also checked to confirm that they belonged to the same cryptographic key pair.

## Nginx Configuration Backup

Before changing the web server configuration, the existing Nginx site was backed up.

Backup:

```text
/etc/nginx/sites-available/default.bak-etapa07
```

The backup was compared with the original before continuing.

## HTTPS Configuration

The active Nginx configuration remained:

```text
/etc/nginx/sites-available/default
```

The relevant configuration became:

```nginx
listen 80 default_server;
listen [::]:80 default_server;

listen 443 ssl;
listen [::]:443 ssl;

ssl_certificate /etc/ssl/web.lab.test/certs/web.lab.test.crt;
ssl_certificate_key /etc/ssl/web.lab.test/private/web.lab.test.key;

root /var/www/html;

server_name web.lab.test;
```

HTTP remained active on port 80.

No automatic HTTP-to-HTTPS redirect was implemented during this stage.

## Configuration Validation

Before reloading Nginx, the configuration was validated with:

```bash
nginx -t
```

The result confirmed:

```text
syntax is ok
test is successful
```

Only after successful validation was Nginx reloaded.

```bash
systemctl reload nginx
```

## Listening Ports

After the reload, Nginx was listening on both HTTP and HTTPS.

```text
0.0.0.0:80
0.0.0.0:443

[::]:80
[::]:443
```

The server therefore provided:

```text
HTTP  -> TCP 80
HTTPS -> TCP 443
```

## Initial TLS Trust Test

Before the internal CA was installed in the client trust stores, HTTPS connectivity already reached the server successfully.

However, clients correctly rejected the certificate because the internal CA was not yet trusted.

Example:

```text
The certificate of 'web.lab.test' is not trusted.
The certificate of 'web.lab.test' doesn't have a known issuer.
```

This distinguished two separate conditions:

```text
TCP/TLS connectivity -> working
Certificate trust    -> not configured yet
```

## OpenSSL Validation

The TLS connection was inspected with OpenSSL.

The server presented:

```text
Subject: web.lab.test
Issuer: LAB Root CA
```

TLS negotiation produced:

```text
Protocol: TLSv1.3
Cipher: TLS_AES_256_GCM_SHA384
```

Without the CA certificate, verification failed because the issuer was unknown.

When the internal CA and hostname verification were explicitly provided, OpenSSL returned:

```text
Verification: OK
Verified peername: web.lab.test
Verify return code: 0 (ok)
```

This confirmed both certificate-chain and hostname validation.

## VM6 Trust Configuration

VM6 was used as an independent Linux client.

Before trusting the CA:

```text
web.lab.test -> 192.168.56.30
TCP 443 -> reachable
Certificate issuer -> not trusted
```

Only the public CA certificate was transferred to VM6.

It was installed as:

```text
/usr/local/share/ca-certificates/lab-root-ca.crt
```

The system certificate store was then updated:

```bash
update-ca-certificates
```

The result reported:

```text
1 added, 0 removed
```

After installation, VM6 successfully accessed:

```text
https://web.lab.test/
```

without manually specifying the CA certificate.

OpenSSL also returned:

```text
Verification: OK
Verified peername: web.lab.test
Protocol: TLSv1.3
Verify return code: 0 (ok)
```

This confirmed that the Debian trust store recognized `LAB Root CA`.

## Firefox on VM6

Firefox on VM6 still reported:

```text
SEC_ERROR_UNKNOWN_ISSUER
```

even after the Debian system trust store accepted the CA.

The system-level TLS validation with `wget` and OpenSSL remained successful.

The browser trust behavior was therefore treated separately from the Debian system certificate store.

## Windows Validation

Windows first confirmed DNS resolution:

```powershell
Resolve-DnsName web.lab.test
```

Result:

```text
web.lab.test -> 192.168.56.30
```

TCP connectivity was then tested:

```powershell
Test-NetConnection web.lab.test -Port 443
```

The result included:

```text
RemoteAddress    : 192.168.56.30
RemotePort       : 443
SourceAddress    : 192.168.56.1
TcpTestSucceeded : True
```

## Windows Trust Before CA Installation

Before the CA was trusted by Windows, HTTPS returned:

```text
SEC_E_UNTRUSTED_ROOT
curl: (60)
```

This was expected because `LAB Root CA` was not yet part of the Windows trusted root store.

## Windows CA Installation

Only the public certificate was transferred to Windows:

```text
lab-ca.crt
```

The CA identity was checked before installation.

It was initially imported into:

```text
Cert:\CurrentUser\Root
```

During later browser validation, the CA was installed with administrator privileges into:

```text
Cert:\LocalMachine\Root
```

After the correct trust configuration, Windows recognized:

```text
LAB Root CA
```

as a trusted root certificate authority.

## Schannel Revocation Behavior

After the CA became trusted, Windows Schannel reported:

```text
CRYPT_E_NO_REVOCATION_CHECK
```

The internal laboratory CA does not publish a Certificate Revocation List (CRL) or provide OCSP.

For diagnostic purposes, the revocation check was isolated using:

```powershell
curl.exe -v --ssl-no-revoke https://web.lab.test/
```

The server returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.22.1
```

`--ssl-no-revoke` was used only to bypass Schannel revocation checking during that test.

Certificate-chain and hostname validation remained active.

## Firefox on Windows

An old manual Firefox certificate exception was removed so that it would not interfere with CA validation.

The browser setting:

```text
security.enterprise_roots.enabled = true
```

was verified.

After `LAB Root CA` was correctly installed in the Windows Local Machine root store and Firefox was restarted:

```text
https://web.lab.test/
```

opened without the previous untrusted-certificate warning and without a manual exception.

## Nginx Logs

The Nginx logs were reviewed after HTTPS testing.

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Successful requests were recorded from:

```text
192.168.56.30  -> VM4
192.168.56.1   -> Windows
192.168.56.111 -> VM6
```

Successful HTTP responses included:

```text
200 OK
304 Not Modified
```

No new TLS-related errors were found in the Nginx error log.

## Security Considerations

The following files must never be published in the Git repository:

```text
/etc/ssl/lab-ca/private/lab-ca.key
/etc/ssl/web.lab.test/private/web.lab.test.key
```

The public CA certificate may be distributed:

```text
/etc/ssl/lab-ca/certs/lab-ca.crt
```

The public server certificate may also be inspected or distributed:

```text
/etc/ssl/web.lab.test/certs/web.lab.test.crt
```

For this laboratory, the Root CA private key was stored on VM4 for educational purposes.

A production PKI should separate the root CA from the web server and protect the root private key independently.

## Current State

The main HTTPS implementation is operational and validated.

Confirmed state:

```text
web.lab.test -> 192.168.56.30

Nginx:
active
enabled

HTTP:
TCP 80 LISTEN

HTTPS:
TCP 443 LISTEN

Certificate:
Subject = web.lab.test
Issuer = LAB Root CA
SAN = DNS:web.lab.test

TLS:
TLS 1.3 validated

Clients:
VM4    -> HTTPS validated
VM6    -> HTTPS validated through Debian trust store
Windows -> TCP/443 and CA trust validated
Firefox on Windows -> trusted without manual exception
```

## Pending Validation

Two items were not completed during the recorded stage.

### Post-reboot validation

VM4 still needs to be rebooted and the following state revalidated:

```text
IPv4 configuration
DNS resolution
nginx active
TCP 80
TCP 443
HTTP
HTTPS
```

Because this reboot validation has not yet been recorded, this stage is considered implemented and functional but not fully closed.

### HTTP to HTTPS Redirect

An automatic redirect from:

```text
http://web.lab.test/
```

to:

```text
https://web.lab.test/
```

was discussed but not implemented.

HTTP and HTTPS currently remain available independently.

## Possible Improvements

Future improvements include:

```text
Dedicated Nginx site file for web.lab.test
Offline or dedicated Root CA
Intermediate Certificate Authority
CRL or OCSP support
Unique functional hostnames for the virtual machines
HTTP to HTTPS redirection
Post-reboot service validation
```

**Status: Implemented — post-reboot validation pending**
