[Voltar](README.md)

## CGNAT

Segue a configuração para CGNAT abordada neste lab
```sql
/interface bridge add name=lo0
/interface wireless security-profiles set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance set [ find default=yes ] redistribute-connected=as-type-1 router-id=10.0.0.3
/ip address add address=10.1.1.1/30 comment=BRAS interface=ether2 network=10.1.1.0
/ip address add address=10.2.1.1/30 comment=BGP interface=ether3 network=10.2.1.0
/ip address add address=10.0.0.3 interface=lo0 network=10.0.0.3
/ip dhcp-client add disabled=no interface=ether1
/ip firewall nat add action=same chain=srcnat comment="nat simbolico" out-interface=ether3 same-not-by-dst=yes src-address=100.64
.0.0/10 to-addresses=198.18.0.0/15
/ip route add check-gateway=ping distance=1 gateway=10.2.1.2
/ip route add check-gateway=ping comment="Rede BRAS" distance=1 dst-address=100.64.0.0/10 gateway=10.1.1.2
/ip route add comment="Rede publica ficticia" distance=1 dst-address=198.18.0.0/15 type=blackhole
/routing ospf interface add interface=ether2 network-type=point-to-point
/routing ospf interface add interface=ether3 network-type=point-to-point
/routing ospf interface add interface=lo0 network-type=broadcast passive=yes
/routing ospf network add area=backbone network=10.1.1.0/30
/routing ospf network add area=backbone network=10.2.1.0/30
/routing ospf network add area=backbone network=10.0.0.3/32
/system identity set name=CGNat
```

Segue a configuração para BGP utilizada neste LAB


```sql
/interface bridge add name=lo0
/interface wireless security-profiles set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance set [ find default=yes ] redistribute-connected=as-type-1 router-id=10.0.0.1
/ip address add address=10.2.1.2/30 comment=CGNAT interface=ether3 network=10.2.1.0
/ip address add address=10.3.1.1/30 comment=BNG interface=ether2 network=10.3.1.0
/ip address add address=10.0.0.1 interface=lo0 network=10.0.0.1
/ip address add address=10.4.1.1/30 comment=Radius interface=ether4 network=10.4.1.0
/ip dhcp-client add disabled=no interface=ether1
/ip firewall nat add action=masquerade chain=srcnat out-interface=ether1
/ip route add check-gateway=ping comment="rede ASN" distance=1 dst-address=198.18.0.0/15 gateway=10.2.1.1
/routing ospf interface add interface=ether2 network-type=point-to-point
/routing ospf interface add interface=ether3 network-type=point-to-point
/routing ospf interface add interface=lo0 network-type=broadcast passive=yes
/routing ospf interface add interface=ether4 network-type=broadcast passive=yes
/routing ospf network add area=backbone network=10.3.1.0/30
/routing ospf network add area=backbone network=10.0.0.1/32
/routing ospf network add area=backbone network=10.2.1.0/30
/routing ospf network add area=backbone network=10.4.1.0/30
/system identity set name=BGP
```
> Note que ambos BGP e CGNAT são meramente simbólicos
