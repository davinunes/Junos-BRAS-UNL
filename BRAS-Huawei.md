Nesse lab é utilizada a imagem AR1000V de Huawei. Já aviso de antemão que a versão do VRP dessa imagem é mais antiga, de modo que alguns comandos mudam completamente de sintaxe.

Ainda assim, resolvi documentar pela experiencia.
Em outro artigo deixarei o lab com um equipamento real.

Já nem preciso dizer que é tenso usar essa imagem no terminal, ela roda muito pesada. Para aceletrar um pouco aumentei a quantidade de nucleos de cpu e memória, ficando com 8G de memoria e 8cpu. 

A primeira coisa a fazer é providenciar acesso ssh, assim podemos deixar de usar o terminal serial que é lerdo e não reconhece a borracha:

> **Dica**: Para usar a borrracha nesse vrp pode ser necessário usar **CTRL + H**

```sql
system-view

ssh user lab authentication-type password
stelnet server enable
ssh server cipher aes256_ctr aes192_ctr
ssh server permit interface GigabitEthernet0/0/0

aaa
 local-user lab password irreversible-cipher lab11223344 
 local-user lab privilege level 15
 local-user lab service-type telnet ssh
 q
#
interface GigabitEthernet0/0/0
 ip address 192.168.4.51 255.255.255.0
#
```

>Logando pela primeira vez em ssh vai te obrigar a mudar a senha, coloquei **Lab123456**

Subir um BRAS/BNG nessa imagem é bem fácil: 

```sql
ip pool pool1
network 100.64.3.0 mask 24
# Esse é o IP que será o Gateway dos clientes
gateway-list 10.255.255.222
q

interface LoopBack 0
 ip address 10.252.250.1 32
 q

interface Virtual-Template1
 ppp authentication-mode pap domain system
 remote address pool pool1
 ppp keepalive retry-times 2
 ppp keepalive in-traffic check
 ppp ipcp dns 8.8.8.8
 timer hold 30
 ip address unnumbered interface LoopBack0

 q

interface GigabitEthernet 0/0/1
 pppoe-server bind virtual-template 1
 q

radius-server template radius
 radius-server authentication 192.168.4.25 1812
 # Sem o comando abaixo a senha vai truncada para o radius
 radius-server shared-key cipher esqueci
 called-station-id mac-format dot-split
 q

radius-server ip-address 192.168.4.25 shared-key cipher esqueci
y

aaa
 authorization-scheme default
 domain system
  authentication-scheme default
  accounting-scheme default
  authorization-scheme default
  radius-server radius
```

Para ver os logs e informações de debug use esses comandos:

```bash
display access-user
display access-user domain system
display access-user username teste1 detail

terminal monitor
terminal debugging
terminal logging

debugging radius all
```

Este é um exemplo de debug completo de uma conexão pppoe autenticada via radius:

```
<Huawei>
Jun 21 2024 16:42:48.477.1+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Recv a msg(Auth req)
<Huawei>
Jun 21 2024 16:42:48.477.2+00:00 Huawei RDS/7/DEBUG:
[RDS(Msg):]Msg type   :authentication request
[RDS(Msg):]UserID     :2907
[RDS(Msg):]Template no:radius
[RDS(Msg):]Authmethod :0(pap)
[RDS(Msg):]SrcMsg   :-
[RDS(Msg):]Bitmap   :ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff
[RDS(Msg):]SerialNo :4294967295
<Huawei>
Jun 21 2024 16:42:48.477.3+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Fill CurrentServer.
<Huawei>
Jun 21 2024 16:42:48.477.4+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Fill CurrentServer,ucAuthServerNum=1
<Huawei>
Jun 21 2024 16:42:48.477.5+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Fill CurrentServer,Authen=::,send=192.168.4.25
<Huawei>
Jun 21 2024 16:42:48.477.6+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] First select Server, select server.
<Huawei>
Jun 21 2024 16:42:48.477.7+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Encoding mac for authen or acct request.
<Huawei>
Jun 21 2024 16:42:48.477.8+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Encode nas-port-id. (peer IP=0.0.0.0)
<Huawei>
Jun 21 2024 16:42:48.477.9+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] encode nas-port-id NEW format. (portStr=slot=0;subslot=0;port=1;vlanid=0;interfaceName=GigabitEthernet0/0/1)
<Huawei>
Jun 21 2024 16:42:48.477.10+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] RD AddOneVendorAtrrToPacket need'nt proc.
<Huawei>
Jun 21 2024 16:42:48.477.11+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] check vlanif and loopback.(loopBckNo=4294967295, sourceVlan=0, loopBackIfIndex=0, vlanIfIndex=4294967295, IsValidVlanif=0)
<Huawei>
Jun 21 2024 16:42:48.477.12+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Get source ip success.(ip address=192.168.4.51)
<Huawei>
Jun 21 2024 16:42:48.477.13+00:00 Huawei RDS/7/DEBUG:
  RADIUS packet: OUT (TotalLen=291)
  (...)
<Huawei>
Jun 21 2024 16:42:48.477.14+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Send a packet(IP:192.168.4.25,Port:1812,Code:authentication request,ID:103 )
<Huawei>
Jun 21 2024 16:42:48.477.15+00:00 Huawei RDS/7/DEBUG:
  RADIUS Send a Packet.
<Huawei>
Jun 21 2024 16:42:48.477.16+00:00 Huawei RDS/7/DEBUG:
  Server Template: 1
  Server IP   : 192.168.4.25
  Protocol: Standard
  Code    : 1
  Len     : 291
  ID      : 103
  [User-Name                          ] [8 ] [teste1]
  [NAS-Port                           ] [6 ] [4096]
  [Service-Type                       ] [6 ] [2]
  [Framed-Protocol                    ] [6 ] [1]
  [Calling-Station-Id                 ] [16] [50ba-0700-3502]
  [NAS-Identifier                     ] [8 ] [Huawei]
  [NAS-Port-Type                      ] [6 ] [15]
  [NAS-Port-Id                        ] [69] [slot=0;subslot=0;port=1;vlanid=0;interfaceName=GigabitEthernet0/0/1]
  [NAS-IP-Address                     ] [6 ] [192.168.4.51]
  [Acct-Session-Id                    ] [35] [Huawei0000100000000068ee1f0000fec]
  [HW-NAS-Startup-Time-Stamp          ] [6 ] [1718938795]
  [HW-IP-Host-Address                 ] [35] [255.255.255.255 50:ba:07:00:35:02]
  [HW-Connect-ID                      ] [6 ] [4076]
<Huawei>
Jun 21 2024 16:42:48.477.17+00:00 Huawei RDS/7/DEBUG:
  [HW-Version                         ] [16] [Huawei AR1000V]
  [HW-Product-ID                      ] [4 ] [AR]
  [HW-Access-Type                     ] [6 ] [7]
  [HW-Domain-Name                     ] [8 ] [system]
<Huawei>
Jun 21 2024 16:42:48.517.1+00:00 Huawei RDS/7/DEBUG:
  RADIUS packet: IN (TotalLen=159)
  (...)
<Huawei>
Jun 21 2024 16:42:48.517.2+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] [RD Compare Global SrcIp]Srcip is not different
<Huawei>
Jun 21 2024 16:42:48.517.3+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] [RD Compare Global SrcIp]Srcip is not different
<Huawei>
Jun 21 2024 16:42:48.517.1+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Process Templat State Chg: Server old state = 1, old up reason = 2, new state = 1, new up reason = 2
<Huawei>
Jun 21 2024 16:42:48.517.2+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Process Templat State Chg: Server old state = 1, old up reason = 2, new state = 1, new up reason = 2
<Huawei>
Jun 21 2024 16:42:48.517.3+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Receive a packet(IP:192.168.4.25,Port:1812,Code:authentication accept,ID:103 )
<Huawei>
Jun 21 2024 16:42:48.517.4+00:00 Huawei RDS/7/DEBUG:
  RADIUS Received a Packet.
<Huawei>
Jun 21 2024 16:42:48.517.5+00:00 Huawei RDS/7/DEBUG:
  Server Template: 1
  Server IP   : 192.168.4.25
  Server Port : 1812
  Protocol: Standard
  Code    : 2
  Len     : 159
  ID      : 103
  [Framed-Route                       ] [25] [192.168.55.0/24 0.0.0.0]
  [Cisco-avpair                       ] [32] [69 70 3a 73 75 62 2d 71 6f 73 2d 70 6f 6c 69 63 79 2d 69 6e 3d 55 50 4c 4f 41 44 32 35 4d ]
  [Cisco-avpair                       ] [35] [69 70 3a 73 75 62 2d 71 6f 73 2d 70 6f 6c 69 63 79 2d 6f 75 74 3d 44 4f 57 4e 4c 4f 41 44 32 35 4d ]
<Huawei>
Jun 21 2024 16:42:48.517.6+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Unknown attr.
<Huawei>
Jun 21 2024 16:42:48.517.7+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Unknown attr.
<Huawei>
Jun 21 2024 16:42:48.517.8+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Supported attr.
<Huawei>
Jun 21 2024 16:42:48.517.9+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Supported attr.
<Huawei>
Jun 21 2024 16:42:48.517.10+00:00 Huawei RDS/7/DEBUG:Attr too long.
(Cisco-avpair(897)).
<Huawei>
Jun 21 2024 16:42:48.517.11+00:00 Huawei RDS/7/DEBUG:
[RDS(Evt):] Send a msg(Auth accept)
<Huawei>
Jun 21 2024 16:42:48.517.12+00:00 Huawei RDS/7/DEBUG:
[RDS(Msg):]Msg type   :authentication accept
[RDS(Msg):]UserID     :2907
[RDS(Msg):]Template no:radius
[RDS(Msg):]SrcMsg   :authentication request
[RDS(Msg):]Bitmap   :00 00 00 00 08 00 00 02 00 00 00 00 00 00 00 00
[RDS(Msg):]SerialNo :4294967295

```


