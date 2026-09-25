# Stage 01 — Infrastructure

## Overview

The first stage of the lab focused on building the virtual infrastructure in Oracle VirtualBox.

The environment uses Windows as the host operating system and six Debian 12 Bookworm virtual machines.

The virtual machines were identified as:

- ROOT
- VM2
- VM3
- VM4
- VM5
- VM6

Each VM was initially configured with 2 GB of RAM and two network interfaces.

## Network Architecture

Two network adapters were configured on each virtual machine, with separate responsibilities.

| Debian Interface | VirtualBox Mode | Purpose |
|---|---|---|
| `enp0s3` | NAT | External and Internet access |
| `enp0s8` | Host-Only | Internal laboratory communication |

## NAT Interface

The `enp0s3` interface uses VirtualBox NAT.

It is responsible for external network access from the virtual machines.

During the lab tests, the NAT gateway was:

`10.0.2.2`

The NAT network remained separate from the internal Host-Only network.

## Host-Only Interface

The second network interface was connected to the VirtualBox Host-Only network.

Internal network:

`192.168.56.0/24`

Windows host address:

`192.168.56.1`

Debian interface:

`enp0s8`

This network became the main internal communication segment for the laboratory.

## Initial Topology

```text
                         Internet
                            |
                     VirtualBox NAT
                            |
                         enp0s3
                            |
        +---------+---------+---------+---------+---------+
        |         |         |         |         |         |
      ROOT       VM2       VM3       VM4       VM5       VM6


                      Windows Host
                     192.168.56.1
                            |
                        Host-Only
                     192.168.56.0/24
                            |
        +---------+---------+---------+---------+---------+
        |         |         |         |         |         |
      ROOT       VM2       VM3       VM4       VM5       VM6
        |         |         |         |         |         |
      enp0s8    enp0s8    enp0s8    enp0s8    enp0s8    enp0s8
