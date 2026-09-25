# VirtualBox Debian Network Lab

Laboratório de redes desenvolvido em Oracle VirtualBox utilizando Debian 12 Bookworm.

O projeto foi construído para implementar e validar uma pequena infraestrutura Linux com serviços separados de rede, administração remota, DNS, DHCP, servidor Web e HTTPS/TLS.

## Ambiente

- Host: Windows
- Hypervisor: Oracle VirtualBox
- Guests: Debian 12 Bookworm
- Rede interna: `192.168.56.0/24`
- NAT: acesso externo das VMs
- Host-Only: comunicação interna do laboratório

## Arquitetura atual

| Máquina | Função | IPv4 Host-Only |
|---|---|---|
| Windows | Administração | `192.168.56.1` |
| ROOT | Administração / cliente | DHCP |
| VM2 | DNS / BIND9 | `192.168.56.20` |
| VM3 | DHCP / Kea DHCP4 | `192.168.56.10` |
| VM4 | Web / Nginx / HTTPS | `192.168.56.30` |
| VM5 | Reservada para expansão | DHCP |
| VM6 | Cliente de testes | DHCP |

Cada VM possui duas interfaces:

- `enp0s3` — NAT
- `enp0s8` — Host-Only

## Serviços implementados

### SSH

Administração das VMs Debian a partir do Windows utilizando OpenSSH.

Foram validados:

- SSH na porta TCP 22
- autenticação por senha
- autenticação por chave pública Ed25519
- `authorized_keys`
- `known_hosts`
- fingerprints dos servidores
- sessões Windows → Debian

### DNS

Servidor DNS executado na VM2:

`192.168.56.20`

Software:

`BIND9`

Domínio interno:

`lab.test`

Registros utilizados:

- `dns.lab.test`
- `dhcp.lab.test`
- `web.lab.test`

### DHCP

Servidor DHCP executado na VM3:

`192.168.56.10`

Software:

`Kea DHCP4`

Pool utilizado:

`192.168.56.101 - 192.168.56.200`

O servidor fornece aos clientes:

- IPv4
- máscara de rede
- DNS interno
- domínio `lab.test`

O DHCP original do VirtualBox foi posteriormente desativado.

### Servidor Web

Servidor Web executado na VM4:

`192.168.56.30`

Software:

`Nginx`

Endereço interno:

`http://web.lab.test/`

DocumentRoot:

`/var/www/html`

### HTTPS / TLS

A VM4 também foi configurada para atender:

`https://web.lab.test/`

Foi criada uma autoridade certificadora interna:

`LAB Root CA`

O certificado do servidor possui:

- CN: `web.lab.test`
- SAN: `DNS:web.lab.test`
- Issuer: `LAB Root CA`

A configuração foi validada com OpenSSL e clientes Windows/Linux.

TLS 1.3 foi negociado durante os testes.

## Etapas

| Etapa | Conteúdo | Estado |
|---|---|---|
| 01 | Infraestrutura | Concluída |
| 02 | Comunicação entre VMs | Concluída |
| 03 | IP, rotas e subnetting | Concluída |
| 04 | SSH | Concluída |
| 05 | DNS + DHCP | Concluída |
| 06 | Nginx / HTTP | Concluída |
| 07 | HTTPS / TLS | Implementada |

## Estrutura do repositório

```text
virtualbox-debian-network-lab/
├── README.md
├── docs/
├── configs/
├── diagrams/
└── evidence/
