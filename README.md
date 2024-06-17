# Junos-BRAS-UNL

## Cenário

Esse projeto inicialmente foi um laboratório para BNG em Juniper, poré, resolvi atualizar com todos os fabricantes que tenho conhecimento. Sempre que for possivel com roteadores virtuais.


Foi utilizada a imagem para eve-ng do junos:  [**17.1R1.8**](https://drive.google.com/drive/folders/11cEkLSjl3mPRLB2FF9Fe0oWUZfxEZKbs?usp=sharing)

Foi utilizada a Imagem **csr1000vng-universalk9.16.03.01** do Cisco CSR

A primeira topologia foi nesse cenário ficticio:
![](img/topologia.png)

Sendo assim, o BGP, a CPE do cliente e CGNAT estão representados por imagens Mikrotik, que farão o trabalho  simplificado para simbolizar os demais atores de uma rede.

Alguns equipamentos estão com uma interface conectada a uma Nuvem chamada **Console**, com a finalidade de facilitar o gerenciamento do equipamento através do emulador.

Como preparação, é preciso inicilizar todos os hosts e então realizar as configurações básicas, a fim de que haja conectividade de internet na VM BGP e que o protocolo ospf esteja rodando entre os 3 Roteadores.

> **Nota:** Inicialmente, testei o LAB utilizando eve-ng, entretanto, na ultima atualização, foi realizado no pnetlab, inclusive as ilustrações são deste ultimo.

## RADIUS

Montagem do **Freeradius** aqui: [FreeRadius](freeradius.md)

User Manager da Mikrotik: [UserMan](userman.md)

## BNGs:

Montagem do **BNG Junos** aqui: [Junos](BRAS-Junos.md)

Montagem do **BNG Cisco** aqui: [Cisco](BRAS-Cisco.md)

Montagem do **BNG Mikrotik** aqui: [Cisco](BRAS-Mikrotik.md)


## Core

- BGP
- OSPF
- GNAT
- LOGs
