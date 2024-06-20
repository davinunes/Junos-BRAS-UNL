
# Configurações iniciais

```php
configure terminal

ip name-server 8.8.8.8
no ip domain lookup

# ajusta data hora e formato de tempo para os logs
ntp server 200.160.0.8
service timestamps log datetime localtime
clock timezone GMT -3
```

### Ativa roteamento IPv6
```php
ipv6 unicast-routing
```


# Configurar SSH
```php
hostname CiscoBNG
aaa new-model
ip domain name davinunes.eti.br
# enable secret cisco
aaa authentication enable default none
username lab privilege 15 password 0 lab
line vty 0 15
transport input ssh
exit
crypto key generate rsa
# Lembrar que a chave deve ser 1024
ip ssh time-out 120
ip ssh authentication-retries 5
ip ssh version 2

interface GigabitEthernet1
 ip address dhcp
 negotiation auto
 exit
```

# Configurar os POOLs de IP
```php
ip local pool bloqueados 10.64.0.0 10.64.3.255
ip local pool cgnat 100.64.0.0 100.64.15.255

ipv6 local pool pool-v6-ndra 2001:DB8:1000::/48 64
ipv6 local pool pool-v6-pd 2001:DB8:8000::/40 56

ipv6 dhcp pool dhcpv6
 prefix-delegation pool pool-v6-pd lifetime 18000 500
 dns-server 2001:4860:4680::8888
 exit
```

# Integração com RADIUS
```php
subscriber templating

aaa new-model

aaa authentication ppp default group radius
aaa authorization network default group radius
aaa authorization subscriber-service default local group radius
aaa accounting delay-start
aaa accounting update periodic 1
aaa accounting network default
 action-type start-stop
 group radius
 exit
aaa nas port extended

aaa server radius dynamic-author
 client 192.168.4.25 server-key esqueci
 auth-type any
 exit

aaa session-id unique
aaa policy interface-config allow-subinterface

radius-server attribute 44 include-in-access-req default-vrf
radius-server attribute 32 include-in-access-req format %h
radius-server attribute nas-port format d
radius-server attribute 31 mac format one-byte delimiter colon upper-case
radius-server attribute 31 send nas-port-detail mac-only
radius-server attribute nas-port-id include vendor-class-id plus remote-id plus circuit-id
radius-server source-ports extended
radius-server retransmit 2
radius-server timeout 3
radius-server unique-ident 34
radius-server key esqueci

radius server default
 address ipv4 192.168.4.25 auth-port 1812 acct-port 1813
 key esqueci
 exit
### Em um roteador de verdade usar tbm o comando: ip accounting-threshould 4000
```

# VIRTUAL-TEMPLATE
```php
interface Loopback0
 ip address 10.255.255.255 255.255.255.255
 ipv6 address 2001:DB8:255::255/128
 exit

interface Virtual-Template1
 ipv6 enable

 ip unnumbered Loopback0
 ipv6 unnumbered Loopback0
  
 ipv6 nd other-config-flag
 ipv6 nd router-preference High
 ipv6 dhcp server dhcpv6
 
 peer default ip address pool cgnat 
 peer default ipv6 pool pool-v6-ndra
 
 ppp authentication pap chap callin default
 ppp accounting default
 ppp ipcp dns 8.8.8.8 8.8.4.4
 no logging event link-status
 !ip tcp adjust-mss 1452
 !mtu 1480
 exit
 ```

# SERVICE PROFILE
```php
bba-group pppoe global
 virtual-template 1
 vendor-tag circuit-id service
 sessions max limit 64000
 sessions per-vc limit 64000
 sessions per-mac limit 1
 sessions per-vlan limit 64000 inner 64000
 sessions auto cleanup
 exit
```

# CONTROLE DE BANDA ESTATICO
```php
policy-map DOWNLOAD25M
 class class-default
 police 25m
 exit
 exit
 exit
policy-map UPLOAD25M
 class class-default
 police 25m
 exit
 exit
 exit
```

> radgroupreply ou radreply
> ```php
> Cisco-AvPair += ip:sub-qos-policy-in=UPLOAD-25M
> Cisco-AvPair += ip:sub-qos-policy-out=DOWNLOAD-25M
> ```

# DEFININDO INTERFACE COMO PPPOE-SERVER

```php
interface GigabitEthernet4
 ipv6 enable
 pppoe enable group global
 no shutdown
 exit

interface GigabitEthernet4.1000
 encapsulation dot1Q 1000
 ipv6 enable
 pppoe enable group global
 exit
 ```
 



