# VirtualBox Debian Network Lab

A network laboratory built with Oracle VirtualBox and Debian 12 Bookworm.

The project implements a small Linux-based infrastructure with separate services for remote administration, DNS, DHCP, Web hosting, and HTTPS/TLS.

## Environment

- Host: Windows
- Hypervisor: Oracle VirtualBox
- Guests: Debian 12 Bookworm
- Internal network: `192.168.56.0/24`
- NAT: external and Internet access
- Host-Only: internal lab communication

## Current Architecture

| Machine | Role | Host-Only IPv4 |
|---|---|---|
| Windows | Administration host | `192.168.56.1` |
| ROOT | Administration / client | DHCP |
| VM2 | DNS / BIND9 | `192.168.56.20` |
| VM3 | DHCP / Kea DHCP4 | `192.168.56.10` |
| VM4 | Web / Nginx / HTTPS | `192.168.56.30` |
| VM5 | Reserved for future expansion | DHCP |
| VM6 | Test client | DHCP |

Each virtual machine uses two network interfaces:

- `enp0s3` — NAT
- `enp0s8` — Host-Only

## Implemented Services

### SSH

Remote administration of the Debian virtual machines from the Windows host using OpenSSH.

The following were validated:

- SSH over TCP port 22
- password authentication
- Ed25519 public key authentication
- `authorized_keys`
- `known_hosts`
- server fingerprints
- Windows-to-Debian SSH sessions

### DNS

DNS server:

`VM2 — 192.168.56.20`

Software:

`BIND9`

Internal domain:

`lab.test`

Main records:

- `dns.lab.test`
- `dhcp.lab.test`
- `web.lab.test`

### DHCP

DHCP server:

`VM3 — 192.168.56.10`

Software:

`Kea DHCP4`

Address pool:

`192.168.56.101 - 192.168.56.200`

The DHCP server provides:

- IPv4 addresses
- subnet mask
- internal DNS server
- `lab.test` search domain

The original VirtualBox DHCP service was later disabled and replaced by Kea.

### Web Server

Web server:

`VM4 — 192.168.56.30`

Software:

`Nginx`

Internal HTTP endpoint:

`http://web.lab.test/`

Document root:

`/var/www/html`

### HTTPS / TLS

VM4 was also configured to provide:

`https://web.lab.test/`

An internal certificate authority was created:

`LAB Root CA`

The server certificate contains:

- Common Name: `web.lab.test`
- Subject Alternative Name: `DNS:web.lab.test`
- Issuer: `LAB Root CA`

The certificate chain and hostname validation were tested using OpenSSL and Windows/Linux clients.

TLS 1.3 was negotiated during the validation tests.

## Project Stages

| Stage | Topic | Status |
|---|---|---|
| 01 | Infrastructure | Completed |
| 02 | Network Connectivity | Completed |
| 03 | IP, Routing and Subnetting | Completed |
| 04 | SSH Remote Administration | Completed |
| 05 | DNS and DHCP | Completed |
| 06 | Nginx / HTTP | Completed |
| 07 | HTTPS / TLS | Implemented |

## Repository Structure

```text
virtualbox-debian-network-lab/
├── README.md
├── docs/
├── configs/
├── diagrams/
└── evidence/
