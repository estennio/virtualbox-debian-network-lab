# Stage 02 — Network Connectivity

## Overview

This stage validated communication between the Windows host and the Debian virtual machines over the VirtualBox Host-Only network.

The internal network used during the tests was:

`192.168.56.0/24`

All VM-to-VM and VM-to-host traffic in this stage used the `enp0s8` interface.

## Addressing During the Tests

The following IPv4 addresses were assigned by DHCP during this stage:

| Machine | Host-Only IPv4 |
|---|---|
| Windows | `192.168.56.1` |
| ROOT | `192.168.56.106` |
| VM2 | `192.168.56.107` |
| VM3 | `192.168.56.108` |
| VM4 | `192.168.56.109` |
| VM5 | `192.168.56.110` |
| VM6 | `192.168.56.111` |

These VM addresses were dynamically assigned and represent the values used during the connectivity tests.

Some server addresses were changed to static values in later stages.

## Connectivity Tests

Connectivity was verified between multiple systems on the Host-Only network.

Tests included:

- ROOT → Windows
- ROOT → VM2
- ROOT → VM3
- ROOT → VM4
- ROOT → VM5
- ROOT → VM6
- VM6 → ROOT

The recorded ping tests returned:

- 4 responses from 4 requests
- 0% packet loss

This confirmed that the virtual machines could communicate directly through the internal network.

## Internal Traffic Path

Traffic destined for another system inside `192.168.56.0/24` did not require a gateway.

Example:

```text
ROOT
192.168.56.106
      |
      | enp0s8
      |
      | 192.168.56.0/24
      |
      v
VM6
192.168.56.111
