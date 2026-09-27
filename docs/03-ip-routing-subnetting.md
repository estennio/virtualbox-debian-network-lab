# Stage 03 — IP, Routing and Subnetting

## Overview

This stage focused on IPv4 addressing, route selection, and subnetting inside the laboratory.

The goal was to verify how the Debian virtual machines distinguish internal traffic from external traffic and which network interface is selected for each destination.

## Network Context

The laboratory uses two network interfaces on each virtual machine:

| Interface | Network Type | Purpose |
|---|---|---|
| `enp0s3` | NAT | External network and Internet access |
| `enp0s8` | Host-Only | Internal laboratory communication |

Internal network:

`192.168.56.0/24`

VirtualBox NAT gateway:

`10.0.2.2`

## Internal Routing

Traffic destined for the internal laboratory network uses the Host-Only interface.

```text
192.168.56.0/24
        |
        v
      enp0s8
