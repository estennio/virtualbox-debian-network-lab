# VirtualBox Debian Network Lab

A Linux network laboratory built with Oracle VirtualBox and Debian 12 Bookworm.

This project simulates a small infrastructure environment with multiple virtual machines providing network services such as SSH, DNS, DHCP, HTTP, HTTPS and an internal Public Key Infrastructure.

## Project Overview

The laboratory was created to practice and document core Linux networking and infrastructure concepts in a controlled virtual environment.

The environment uses:

- Windows as the host operating system
- Oracle VirtualBox as the hypervisor
- Debian 12 Bookworm as the guest operating system
- NAT for external and Internet access
- Host-Only networking for internal laboratory communication

The internal network is:

```text
192.168.56.0/24
```

## Architecture

| Machine | Role | Host-Only IPv4 |
|---|---|---|
| Windows Host | Administration | `192.168.56.1` |
| ROOT | Administration / client | DHCP |
| VM2 | DNS / BIND9 | `192.168.56.20` |
| VM3 | DHCP / Kea DHCP4 | `192.168.56.10` |
| VM4 | Web / Nginx / HTTPS | `192.168.56.30` |
| VM5 | Reserved for future expansion | DHCP |
| VM6 | Test client | DHCP |

Each Debian virtual machine uses two network interfaces:

```text
enp0s3 -> NAT
enp0s8 -> Host-Only
```

The NAT interface provides external connectivity.

The Host-Only interface is used for communication between the Windows host and the Debian virtual machines.

## Implemented Services

### SSH

Remote administration was implemented using OpenSSH.

The laboratory validates:

- TCP port 22 connectivity
- password authentication
- Ed25519 public key authentication
- `authorized_keys`
- `known_hosts`
- SSH host fingerprints
- remote administration from Windows
- SSH logging and troubleshooting

### DNS

DNS is provided by:

```text
VM2
192.168.56.20
BIND9
```

Internal domain:

```text
lab.test
```

Main records include:

```text
dns.lab.test
dhcp.lab.test
web.lab.test
```

### DHCP

DHCP is provided by:

```text
VM3
192.168.56.10
Kea DHCP4
```

The DHCP server provides clients with:

- IPv4 configuration
- subnet mask
- internal DNS server
- `lab.test` search domain

The original VirtualBox Host-Only DHCP service was replaced by Kea.

### Web Server

The internal web server runs on:

```text
VM4
192.168.56.30
Nginx
```

HTTP endpoint:

```text
http://web.lab.test/
```

Document root:

```text
/var/www/html
```

### HTTPS / TLS

HTTPS was added to the Nginx server using an internal Public Key Infrastructure.

HTTPS endpoint:

```text
https://web.lab.test/
```

Internal certificate authority:

```text
LAB Root CA
```

The server certificate includes:

```text
Common Name: web.lab.test
Subject Alternative Name: DNS:web.lab.test
Issuer: LAB Root CA
```

TLS 1.3 was successfully validated during the laboratory tests.

Certificate trust was tested on Debian and Windows clients.

## Project Stages

| Stage | Topic | Status |
|---|---|---|
| 01 | Infrastructure | Completed |
| 02 | Network Connectivity | Completed |
| 03 | IP, Routing and Subnetting | Completed |
| 04 | SSH Remote Administration | Completed |
| 05 | DNS and DHCP | Completed |
| 06 | Nginx / HTTP | Completed |
| 07 | HTTPS / TLS / Internal PKI | Implemented — final reboot validation pending |

## Documentation

Detailed documentation is available for each stage:

- [01 — Infrastructure](docs/01-infrastructure.md)
- [02 — Network Connectivity](docs/02-network-connectivity.md)
- [03 — IP, Routing and Subnetting](docs/03-ip-routing-subnetting.md)
- [04 — SSH Remote Administration](docs/04-ssh-remote-administration.md)
- [05 — DNS and DHCP](docs/05-dns-dhcp.md)
- [06 — Nginx HTTP](docs/06-nginx-http.md)
- [07 — HTTPS, TLS and Internal PKI](docs/07-https-tls.md)

## Repository Structure

```text
virtualbox-debian-network-lab/
├── README.md
├── docs/
│   ├── 01-infrastructure.md
│   ├── 02-network-connectivity.md
│   ├── 03-ip-routing-subnetting.md
│   ├── 04-ssh-remote-administration.md
│   ├── 05-dns-dhcp.md
│   ├── 06-nginx-http.md
│   └── 07-https-tls.md
├── configs/
├── diagrams/
└── evidence/
```

The `configs`, `diagrams` and `evidence` directories are intended for sanitized configuration examples, topology diagrams and validation evidence.

## Network Flow

The laboratory separates internal and external traffic.

```text
                         Internet
                            |
                      VirtualBox NAT
                            |
                          enp0s3
                            |
        +---------+---------+---------+---------+
        |         |         |         |         |
      ROOT       VM2       VM3       VM4       VM6
                  |         |         |
                BIND9      Kea      Nginx
                  |         |       HTTP/HTTPS
                  |
                DNS
                            |
                     Host-Only Network
                      192.168.56.0/24
                            |
                      Windows Host
                      192.168.56.1
```

## Security Notes

Private cryptographic material must never be committed to the repository.

Examples of files that must remain private:

```text
*.key
id_ed25519
id_rsa
```

The internal root CA private key must also remain outside the public repository.

Only sanitized configuration files, public certificates and non-sensitive validation evidence should be published.

## Technologies

The laboratory currently includes:

- Oracle VirtualBox
- Debian 12 Bookworm
- TCP/IP
- IPv4
- subnetting
- routing
- SSH
- OpenSSH
- BIND9
- Kea DHCP4
- Nginx
- HTTP
- HTTPS
- TLS
- X.509 certificates
- OpenSSL
- internal PKI
- PowerShell
- Git
- GitHub

## Future Improvements

Possible future improvements include:

- publishing sanitized service configuration files
- creating a graphical network topology diagram
- adding selected validation screenshots
- assigning unique Linux hostnames to every VM
- HTTP to HTTPS redirection
- dedicated Nginx virtual host configuration
- stronger SSH hardening
- firewall rules
- monitoring
- centralized logging
- dedicated or offline certificate authority

## Purpose

This repository documents the practical implementation and troubleshooting of a small Linux network infrastructure.

The goal is not to reproduce a production environment, but to build a controlled laboratory for learning networking, Linux administration, infrastructure services and security concepts.