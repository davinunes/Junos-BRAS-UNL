Usando imagem para eve-ng do vrp1000

Já nem preciso dizer que é tenso usar essa imagem no terminal, então a primeira coisa a fazer é providenciar acesso ssh:

https://support.huawei.com/enterprise/en/doc/EDOC1000178166/3467af6a/configuring-an-ssh-user

https://support.huawei.com/enterprise/en/doc/EDOC1100033773/f2f0ee0a/example-for-configuring-the-pppoe-server#dc_cfg_pppoe_1037

> Para usar a borrracha nesse vrp pode ser necessário usar CTRL + H

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

```sql
ip pool pool1
network 100.64.3.0 mask 24
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