# Etapa 01 — Infraestrutura

## Visão geral

A primeira etapa do laboratório consistiu na criação da infraestrutura virtual no Oracle VirtualBox.

O ambiente utiliza Windows como sistema host e seis máquinas virtuais com Debian 12 Bookworm.

As VMs foram identificadas no VirtualBox como:

- ROOT
- VM2
- VM3
- VM4
- VM5
- VM6

Cada máquina foi configurada inicialmente com 2 GB de memória RAM e duas interfaces de rede.

## Arquitetura de rede

Foram utilizados dois adaptadores por máquina virtual, cada um com uma função específica.

| Interface | Tipo no VirtualBox | Função |
|---|---|---|
| `enp0s3` | NAT | Acesso externo e Internet |
| `enp0s8` | Host-Only | Comunicação interna do laboratório |

### NAT

A interface `enp0s3` foi mantida em modo NAT.

Ela é responsável pelo acesso externo das máquinas virtuais.

Durante os testes do laboratório, o gateway da rede NAT foi:

`10.0.2.2`

A rede NAT permaneceu separada da rede utilizada para comunicação interna entre as VMs.

### Host-Only

A segunda interface foi conectada à rede Host-Only do VirtualBox.

Rede utilizada:

`192.168.56.0/24`

Endereço do host Windows:

`192.168.56.1`

Interface utilizada nas VMs:

`enp0s8`

Essa rede passou a ser o segmento interno principal do laboratório.

## Topologia inicial

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


                     Host Windows
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
