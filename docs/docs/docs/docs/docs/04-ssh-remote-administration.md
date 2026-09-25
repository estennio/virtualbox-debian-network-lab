# Stage 04 — SSH Remote Administration

## Overview

This stage implemented remote administration of the Debian virtual machines from the Windows host using OpenSSH.

SSH access was validated on all six Debian VMs through the VirtualBox Host-Only network.

Two authentication methods were tested:

- password authentication;
- Ed25519 public key authentication.

The final objective was to administer the virtual machines from Windows without depending on the VirtualBox console.

## Environment

| Component | Configuration |
|---|---|
| Administration host | Windows |
| SSH client | OpenSSH for Windows |
| Virtual machines | Debian 12 Bookworm |
| SSH server | OpenSSH Server |
| Internal network | `192.168.56.0/24` |
| Windows Host-Only IPv4 | `192.168.56.1` |
| SSH port | TCP `22` |
| Debian user | `estennio` |

The Windows host connects directly to each VM through the Host-Only network.

ROOT is not used as an SSH jump host.

## Addresses Used During Validation

The following DHCP addresses were used during this stage:

| Machine | Host-Only IPv4 |
|---|---|
| ROOT | `192.168.56.106` |
| VM2 | `192.168.56.107` |
| VM3 | `192.168.56.108` |
| VM4 | `192.168.56.109` |
| VM5 | `192.168.56.110` |
| VM6 | `192.168.56.111` |

These addresses represent the state of the lab during the SSH tests and were not permanent reservations.

Some infrastructure servers received static addresses in later stages.

## OpenSSH Server

`openssh-server` was installed on the Debian machines.

The SSH service was verified using:

```bash
systemctl status ssh
```

The service was confirmed as:

```text
active
enabled
```

The SSH daemon was also verified listening on TCP port 22.

Example:

```bash
ss -lntp 'sport = :22'
```

The observed listeners included:

```text
0.0.0.0:22
[::]:22
```

## TCP Connectivity Validation

Before testing authentication, TCP connectivity from Windows to the Debian machines was verified.

Example:

```powershell
Test-NetConnection -ComputerName 192.168.56.111 -Port 22
```

A successful test returned:

```text
TcpTestSucceeded : True
```

This confirmed that Windows could establish a TCP connection to the SSH service.

A successful TCP test alone does not confirm user authentication.

## Password Authentication

Initial SSH access was tested using the Debian account:

```text
estennio
```

Example connection:

```powershell
ssh estennio@192.168.56.111
```

Password authentication was validated on all six virtual machines.

After login, the remote session could be identified using:

```bash
printenv SSH_CONNECTION
```

This provided the client address, client port, server address, and SSH server port.

The Windows client address observed during the tests was:

`192.168.56.1`

## Public Key Authentication

After password-based access was validated, an Ed25519 key pair was created on Windows specifically for the laboratory.

Key name:

```text
id_ed25519_lab
```

Key type:

```text
ED25519
```

Comment:

```text
windows-lab
```

The private key remained on the Windows host.

Only the public key was copied to the Debian virtual machines.

## Authorized Keys

The public key was installed for the `estennio` account in:

```text
/home/estennio/.ssh/authorized_keys
```

The resulting permissions were verified as:

```text
~/.ssh
700
```

and:

```text
~/.ssh/authorized_keys
600
```

The files were owned by the `estennio` user.

The private key was never copied to the Debian servers.

## Key-Based Login

Because the laboratory key uses a custom filename, it was explicitly selected from PowerShell.

Example:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519_lab" `
    -o IdentitiesOnly=yes `
    -o PreferredAuthentications=publickey `
    estennio@192.168.56.111
```

Using:

```text
PreferredAuthentications=publickey
```

allowed the test to verify public key authentication without falling back to the Debian account password.

Key-based authentication was successfully validated on all six VMs.

## SSH Trust Files

Two different SSH trust mechanisms were used.

### Client-side server identity

Windows maintains known server identities in:

```text
%USERPROFILE%\.ssh\known_hosts
```

This file stores public host keys presented by SSH servers.

It can be used to detect unexpected changes in server identity.

### Server-side user authorization

Each Debian server maintains the user's authorized public keys in:

```text
/home/estennio/.ssh/authorized_keys
```

These files serve different purposes:

| File | Purpose |
|---|---|
| `known_hosts` | Verifies the identity of SSH servers |
| `authorized_keys` | Defines which public keys may authenticate a user |

## SSH Session Validation

The sessions were inspected using:

```bash
printenv SSH_CONNECTION
```

Example from VM6:

```text
192.168.56.1 41512 192.168.56.111 22
```

This showed:

```text
Client: 192.168.56.1
Server: 192.168.56.111
Server port: 22
```

The client-side source port is temporary and changes between connections.

The SSH session was terminated normally using:

```bash
exit
```

which returned control to Windows PowerShell.

## Remote Administration

Once connected through SSH, standard Linux administration commands could be executed remotely.

Examples used during validation included:

```bash
id
hostname
ip -br -4 addr
free -h
df -h
```

SSH logs were inspected with administrative privileges using:

```bash
journalctl -u ssh
```

On ROOT, the regular `estennio` account was separate from the administrative `root` account.

Administrative access during testing was obtained with:

```bash
su -
```

when required.

## SSH Configuration Observed

The effective SSH configuration was inspected on ROOT with:

```bash
/usr/sbin/sshd -T
```

Relevant values observed were:

| Directive | Value |
|---|---|
| `port` | `22` |
| `permitrootlogin` | `without-password` |
| `pubkeyauthentication` | `yes` |
| `passwordauthentication` | `yes` |
| `authorizedkeysfile` | `.ssh/authorized_keys .ssh/authorized_keys2` |

No SSH port change or major `sshd_config` hardening was performed during this stage.

The objective was to establish and validate remote administration before changing the default SSH policy.

## Troubleshooting

Several issues were identified during the stage.

### PowerShell command executed inside Debian

`Test-NetConnection` was initially executed in a Debian shell.

The command belongs to PowerShell and was therefore moved back to the Windows terminal.

### Linux paths used from Windows

Paths under:

```text
/etc/ssh/
```

belong to the Debian systems and cannot be inspected as Windows filesystem paths.

The relevant files were checked from the correct Debian console or SSH session.

### Incorrect connection origin

One early SSH test on ROOT originated from the VM itself instead of Windows.

The connection was closed and repeated from Windows.

The new session confirmed:

```text
192.168.56.1
```

as the client address.

### Authentication test on VM2

An earlier key authentication attempt on VM2 was rejected.

A later test explicitly restricted to public key authentication succeeded without requiring a server configuration change.

The exact cause of the earlier failure was not established.

### SSH logs and privileges

Some SSH log information was not available to the regular account.

The logs were successfully inspected after switching to the administrative account.

## Hostname Observation

All Debian virtual machines used the hostname:

```text
debian01
```

during this stage.

The VirtualBox names `ROOT`, `VM2`, `VM3`, `VM4`, `VM5`, and `VM6` were therefore used together with their IP addresses to distinguish the machines.

Unique Linux hostnames remain a possible future improvement.

## Security Considerations

The private SSH key must not be committed to this repository.

Files such as the following must remain private:

```text
id_ed25519_lab
id_rsa
*.key
```

Only public keys and sanitized configuration examples should be included in the repository.

Before future changes to the SSH server configuration, the configuration can be validated with:

```bash
/usr/sbin/sshd -t
```

before reloading the service.

## Final State

At the end of this stage:

- `openssh-server` was installed and running on all six Debian VMs;
- TCP port 22 was reachable from Windows;
- password authentication was validated;
- Ed25519 public key authentication was validated;
- the private key remained on Windows;
- the public key was installed in `authorized_keys`;
- server identities were recorded through `known_hosts`;
- SSH sessions were verified as originating from `192.168.56.1`;
- Windows became the primary administration point for the Debian laboratory.

**Status: Completed**
