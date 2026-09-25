# Stage 06 — Nginx HTTP Server

## Overview

This stage introduced the first application service in the laboratory by deploying an Nginx web server on VM4.

VM4 was assigned a static Host-Only address and integrated with the DNS and DHCP infrastructure configured in the previous stage.

The final HTTP service became available through both the server IP address and the internal DNS name:

```text
http://192.168.56.30/
http://web.lab.test/
```

## Server Role

VM4 was assigned the web server role.

| Parameter | Value |
|---|---|
| Machine | VM4 |
| Operating system | Debian 12 Bookworm |
| Service | Nginx |
| Host-Only IPv4 | `192.168.56.30/24` |
| DNS server | `192.168.56.20` |
| DNS search domain | `lab.test` |
| Internal hostname | `web.lab.test` |
| HTTP port | TCP `80` |

The NAT interface remained available for external network access.

## Network Configuration

The Host-Only interface on VM4 was changed from DHCP to a static IPv4 configuration.

Relevant configuration:

```text
Interface: enp0s8
Address: 192.168.56.30/24
DNS: 192.168.56.20
Search domain: lab.test
Gateway: none
```

No gateway was configured on the Host-Only interface.

External connectivity continued through:

```text
enp0s3 -> VirtualBox NAT -> 10.0.2.2
```

This preserved the separation between internal laboratory traffic and Internet access.

## DNS Integration

A DNS record was added to the internal `lab.test` zone:

```text
web.lab.test -> 192.168.56.30
```

Name resolution was validated with:

```bash
getent hosts web.lab.test
```

Result:

```text
192.168.56.30 web.lab.test
```

This allowed clients to access the server using a DNS name instead of its IPv4 address.

## Nginx Installation

Nginx was installed on VM4 and configured to start automatically with the system.

The service was verified as:

```text
active
enabled
```

The relevant systemd service is:

```text
nginx.service
```

The Nginx version recorded during the project was:

```text
1.22.1
```

## HTTP Listener

The web server was verified listening on TCP port 80.

Observed listeners included:

```text
0.0.0.0:80
[::]:80
```

This confirmed that Nginx was accepting HTTP connections over both IPv4 and IPv6 locally.

The validated client traffic in this laboratory used IPv4.

## Document Root

The Nginx document root used during this stage was:

```text
/var/www/html
```

The main page was:

```text
/var/www/html/index.html
```

The default Nginx page was replaced with a simple laboratory page.

The page identified:

```text
Servidor Web do Laboratório
VM4 - Nginx
IP: 192.168.56.30
Domínio: web.lab.test
```

## Nginx Site Configuration

The enabled site was linked through:

```text
/etc/nginx/sites-enabled/default
```

to:

```text
/etc/nginx/sites-available/default
```

At this stage, the active configuration provided HTTP service on TCP port 80.

HTTPS had not yet been configured.

## Connectivity Validation

Windows was used to verify that VM4 was reachable on TCP port 80.

Example:

```powershell
Test-NetConnection 192.168.56.30 -Port 80
```

Successful TCP connectivity confirmed that the service was reachable through the Host-Only network.

## HTTP Validation by IP Address

The HTTP service was tested directly using the server IPv4 address:

```text
http://192.168.56.30/
```

The server returned the Nginx laboratory page successfully.

## HTTP Validation by DNS Name

After DNS integration, the service was also tested from Windows using:

```powershell
curl.exe http://web.lab.test/
```

The response returned the custom laboratory HTML page.

This validated the complete path:

```text
Windows
   |
   | DNS query
   v
VM2 / BIND9
192.168.56.20
   |
   | web.lab.test -> 192.168.56.30
   v
Windows
   |
   | HTTP / TCP 80
   v
VM4 / Nginx
192.168.56.30
```

## Client Validation

The web service was successfully accessed from multiple systems in the laboratory.

The recorded environment confirmed access from:

```text
Windows
VM6
Firefox
```

This demonstrated that the service was not limited to local access on VM4.

## HTTP Status Codes Observed

During testing and log inspection, the Nginx environment recorded several HTTP status codes.

| Status | Meaning in the tests |
|---|---|
| `200 OK` | Successful request |
| `304 Not Modified` | Cached resource remained valid |
| `404 Not Found` | Requested resource did not exist |
| `403 Forbidden` | Access to a tested path was denied |

These responses were used to inspect normal web server behavior and troubleshooting information.

## Nginx Logs

The main Nginx logs used during testing were:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

The access log confirmed requests from laboratory clients, including Windows.

The error log was inspected when troubleshooting unsuccessful HTTP requests.

## Service Architecture

At the end of the stage, the HTTP request path was:

```text
Client
   |
   | DNS
   v
VM2
192.168.56.20
BIND9
   |
   | resolves web.lab.test
   v
192.168.56.30
   |
   | TCP 80 / HTTP
   v
VM4
Nginx
   |
   v
/var/www/html/index.html
```

The DHCP service remained independent on VM3:

```text
VM3
192.168.56.10
Kea DHCP4
```

This maintained the separation of infrastructure roles across different virtual machines.

## Final State

At the end of this stage:

- VM4 had the static Host-Only address `192.168.56.30`;
- Nginx was installed and running;
- Nginx was enabled at boot;
- TCP port 80 was listening;
- the document root was `/var/www/html`;
- the default web page was replaced with a laboratory page;
- `web.lab.test` resolved to `192.168.56.30`;
- HTTP worked through both the IPv4 address and DNS name;
- Windows could reach the service;
- VM6 could reach the service;
- browser access was validated;
- Nginx access and error logs were inspected;
- HTTP was ready to be extended with TLS in the next stage.

**Status: Completed**
