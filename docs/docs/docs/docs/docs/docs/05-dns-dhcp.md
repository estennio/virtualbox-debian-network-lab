# Stage 05 — DNS and DHCP

## Overview

This stage replaced the default VirtualBox DHCP service with dedicated DNS and DHCP services inside the Host-Only network.

The final architecture assigned separate infrastructure roles to two Debian virtual machines:

| Machine | Role | Host-Only IPv4 |
|---|---|---|
| VM2 | DNS / BIND9 | `192.168.56.20` |
| VM3 | DHCP / Kea DHCP4 | `192.168.56.10` |

The internal domain used by the laboratory is:

```text
lab.test
```

The objective was to provide predictable infrastructure addresses while keeping client systems dynamically configured.

## Network Architecture

The existing dual-interface design was preserved.

| Interface | Purpose |
|---|---|
| `enp0s3` | NAT / Internet access |
| `enp0s8` | Host-Only / internal services |

Internal network:

```text
192.168.56.0/24
```

Windows administration host:

```text
192.168.56.1
```

The NAT interface remained responsible for external connectivity.

The Host-Only interface became the network used by the internal DNS and DHCP services.

## Infrastructure Addressing

VM2 and VM3 were changed from dynamic addresses to static Host-Only addresses.

### VM2 — DNS

```text
192.168.56.20/24
```

### VM3 — DHCP

```text
192.168.56.10/24
```

No Host-Only default gateway was configured for these services.

External traffic continued to use the NAT interface and the VirtualBox NAT gateway:

```text
10.0.2.2
```

## DNS Server — VM2

VM2 was configured as the internal DNS server using BIND9.

Configuration summary:

| Parameter | Value |
|---|---|
| Software | BIND9 |
| systemd service | `named.service` |
| Host-Only IPv4 | `192.168.56.20` |
| Internal domain | `lab.test` |
| Service state | `active` |
| Boot state | `enabled` |

## Forward DNS Zone

The forward zone was created for:

```text
lab.test
```

Zone file:

```text
/etc/bind/db.lab.test
```

Records validated during this stage included:

```text
dns.lab.test    A    192.168.56.20
dhcp.lab.test   A    192.168.56.10
```

## Reverse DNS Zone

A reverse DNS zone was also configured.

Zone:

```text
56.168.192.in-addr.arpa
```

Zone file:

```text
/etc/bind/db.192.168.56
```

Validated mappings included:

```text
192.168.56.20 -> dns.lab.test
192.168.56.10 -> dhcp.lab.test
```

This provided both forward and reverse name resolution for the main infrastructure servers.

## DNS Forwarding

BIND9 was configured to forward external DNS queries to:

```text
8.8.4.4
```

This allowed clients to use VM2 as their primary DNS server for both internal and external resolution.

The resulting DNS path was:

```text
Client
   |
   v
VM2 / BIND9
192.168.56.20
   |
   +---- lab.test --------> Internal zone
   |
   +---- external names --> 8.8.4.4
```

## DNS Validation

Direct DNS queries were tested against VM2.

Example:

```bash
dig @192.168.56.20 dns.lab.test A
```

Expected and validated result:

```text
dns.lab.test -> 192.168.56.20
```

Normal client resolution was also tested.

```bash
getent hosts dns.lab.test
```

Result:

```text
192.168.56.20 dns.lab.test
```

The DHCP server name was also resolved:

```bash
getent hosts dhcp.lab.test
```

Result:

```text
192.168.56.10 dhcp.lab.test
```

External resolution was tested using:

```bash
getent ahostsv4 debian.org
```

Public IPv4 addresses were returned successfully.

## DHCP Server — VM3

VM3 was configured as the dedicated DHCP server using Kea DHCP4.

Configuration summary:

| Parameter | Value |
|---|---|
| Software | Kea DHCP4 |
| Service | `kea-dhcp4-server` |
| Host-Only IPv4 | `192.168.56.10` |
| Interface | `enp0s8` |
| Service state | `active` |
| Boot state | `enabled` |

Main configuration file:

```text
/etc/kea/kea-dhcp4.conf
```

## DHCP Scope

The Kea configuration used the following parameters:

| Parameter | Value |
|---|---|
| Network | `192.168.56.0/24` |
| Pool | `192.168.56.101 - 192.168.56.200` |
| Lease time | `600` seconds |
| Renew timer | `300` seconds |
| Rebind timer | `525` seconds |
| Authoritative | `true` |

The DHCP service provided clients with:

```text
DNS server: 192.168.56.20
Domain: lab.test
Subnet mask: 255.255.255.0
```

A Host-Only gateway was intentionally not distributed.

Internet traffic continued to use:

```text
enp0s3 -> NAT -> 10.0.2.2
```

## DHCP Leases

Kea stored IPv4 leases in:

```text
/var/lib/kea/kea-leases4.csv
```

An auxiliary lease file was also observed:

```text
/var/lib/kea/kea-leases4.csv.2
```

Leases were confirmed for multiple laboratory clients, including ROOT, VM4, VM5, and VM6.

One recorded allocation for ROOT was:

```text
192.168.56.106
```

Kea logs confirmed that the address was allocated for the configured 600-second lease period.

## VirtualBox DHCP Migration

Before this stage, DHCP for the Host-Only network was provided by VirtualBox.

Original configuration:

| Parameter | Value |
|---|---|
| DHCP server | `192.168.56.100` |
| Pool | `192.168.56.101 - 192.168.56.254` |
| Mask | `255.255.255.0` |

The VirtualBox DHCP service was disabled after Kea was configured.

The reported state became:

```text
Enabled: No
```

## Duplicate DHCP Server Issue

A problem was identified during the migration.

Although VirtualBox reported its DHCP server as disabled, VM6 was still receiving:

```text
dhcp_server_identifier = 192.168.56.100
```

At the same time, Kea was active at:

```text
192.168.56.10
```

This resulted in two DHCP servers temporarily operating on the same Host-Only network.

The Windows host still had active processes named:

```text
VBoxNetDHCP.exe
```

VirtualBox logs also showed the old DHCP service continuing to send DHCP acknowledgements.

## Resolution

The remaining `VBoxNetDHCP` processes associated with the Host-Only network were identified and terminated.

The process state was checked using PowerShell:

```powershell
Get-Process VBoxNetDHCP -ErrorAction SilentlyContinue
```

After cleanup, no active process was returned.

VirtualBox continued to report:

```text
Enabled: No
```

VM6 then received the expected DHCP information:

```text
dhcp_server_identifier = 192.168.56.10
domain_name_servers = 192.168.56.20
domain_name = lab.test
```

This confirmed that Kea had become the active DHCP server for the laboratory network.

## DNS Priority Issue

Another issue appeared because each VM still had two network interfaces.

The NAT interface also supplied DNS servers, while Kea supplied the internal DNS server.

Some clients initially had resolver entries similar to:

```text
nameserver 8.8.4.4
nameserver fd17:625c:f037:2::3
nameserver 192.168.56.20
```

With the external DNS servers having higher priority, internal names such as:

```text
dns.lab.test
```

could fail during normal resolution.

## DNS Priority Adjustment

The NetworkManager Host-Only profiles were adjusted on:

```text
ROOT
VM4
VM5
VM6
```

The DNS priority was configured as:

```text
ipv4.dns-priority = -50
```

The relevant NetworkManager profile was:

```text
Wired connection 1
```

After reactivating the connection, the resolver configuration became:

```text
search lab.test
nameserver 192.168.56.20
```

The clients therefore used VM2 for normal DNS resolution.

External names remained available because BIND9 forwarded those queries upstream.

## Final Service Flow

### DHCP

```text
Client
   |
   | DHCP
   v
VM3
192.168.56.10
Kea DHCP4
```

### DNS

```text
Client
   |
   | DNS
   v
VM2
192.168.56.20
BIND9
   |
   +---- Internal: lab.test
   |
   +---- External: 8.8.4.4
```

### Internet

```text
Client
   |
   v
enp0s3
   |
   v
VirtualBox NAT
   |
   v
10.0.2.2
   |
   v
Internet
```

## Final State

At the end of this stage:

- VM2 operated as the DNS server at `192.168.56.20`;
- VM3 operated as the DHCP server at `192.168.56.10`;
- BIND9 was active and enabled at boot;
- Kea DHCP4 was active and enabled at boot;
- `lab.test` provided the internal DNS namespace;
- forward and reverse DNS zones were operational;
- clients received IPv4 addresses from Kea;
- clients received `192.168.56.20` as their DNS server;
- clients received `lab.test` as their search domain;
- internal DNS resolution worked;
- external DNS resolution worked through the BIND forwarder;
- the VirtualBox DHCP service was disabled;
- remaining `VBoxNetDHCP` processes were removed from the active network path;
- DNS priority was adjusted so the internal resolver was used correctly.

**Status: Completed**
