# Настроить GRE между офисами Москва и Санкт-Петербург, а также DMVPN между Москвой, Чокурдахом и Лабытнанги.

# План работ:

1. Настроить GRE между офисами Москва и С.-Петербург.
2. Настроить DMVMN между Москва и Чокурдах, Лабытнанги.
3. Все офисы в лабораторной работе должны иметь IP связность.

### Настроим GRE тунели между R14 и R18, а так же R15 и R18.

~~~
R14#
interface Tunnel1
 ip address 10.11.0.5 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 85.123.45.18
 tunnel destination 48.81.46.10
~~~
~~~
R15#
interface Tunnel0
 ip address 10.11.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 185.15.145.54
 tunnel destination 56.23.124.53
~~~
~~~
R18#
interface Tunnel0
 ip address 10.11.0.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 56.23.124.53
 tunnel destination 185.15.145.54
!
interface Tunnel1
 ip address 10.11.0.6 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 48.81.46.10
 tunnel destination 85.123.45.18
~~~

Туннели поднялись.
~~~
R18(config)#do sh ip int brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                172.20.254.5    YES NVRAM  up                    up
Ethernet0/1                172.20.254.2    YES NVRAM  up                    up
Ethernet0/2                56.23.124.53    YES NVRAM  up                    up
Ethernet0/3                48.81.46.10     YES NVRAM  up                    up
Loopback0                  172.20.255.18   YES NVRAM  up                    up
NVI0                       172.20.254.5    YES unset  up                    up
Tunnel0                    10.11.0.2       YES manual up                    up
Tunnel1                    10.11.0.6       YES manual up                    up
~~~
### Настроим DMVPN между Москвой, Чокурдах, Лабытнанги.
Москва центр, R14 и R15 Hubs, Лабытнанги(R27) и Чокурдах(R28) Spoke.
Настроим R14 и R15
```
R15#
interface Tunnel3
 description DMVPN Cloud 1 - Primary HUB
 ip address 10.0.252.1 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 100
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
```
```
R14#
interface Tunnel3
 description DMVPN Cloud 1 - Secondary HUB
 ip address 10.0.252.2 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id 100
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/2
 tunnel mode gre multipoint
```
Настроим SPOKE, R27 и R28
```
R27#
interface Tunnel0
 description DMVPN to Hub
 ip address 10.0.252.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.0.252.1 185.15.145.54
 ip nhrp map multicast 185.15.145.54
 ip nhrp map 10.0.252.2 85.123.45.18
 ip nhrp map multicast 85.123.45.18
 ip nhrp network-id 100
 ip nhrp nhs 10.0.252.1 priority 1 cluster 1
 ip nhrp nhs 10.0.252.2 priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```
```
R28#
interface Tunnel0
 description DMVPN to Hub
 ip address 10.0.252.4 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.0.252.1 185.15.145.54
 ip nhrp map multicast 185.15.145.54
 ip nhrp map 10.0.252.2 85.123.45.18
 ip nhrp map multicast 85.123.45.18
 ip nhrp network-id 100
 ip nhrp nhs 10.0.252.1 priority 1 cluster 1
 ip nhrp nhs 10.0.252.2 priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
```
Туннели поднялись.
```
R15#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel3, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 96.254.180.226       10.0.252.3    UP 00:29:13     D
     1 15.67.83.114         10.0.252.4    UP 00:28:25     D

R15#show ip nhrp detail
10.0.252.3/32 via 10.0.252.3
   Tunnel3 created 00:30:31, expire 01:55:37
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 96.254.180.226
10.0.252.4/32 via 10.0.252.4
   Tunnel3 created 00:30:55, expire 01:30:18
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 15.67.83.114

```
```
R14#sh dmvpn
Legend: Attrb --> S - Static, D - Dynamic, I - Incomplete
        N - NATed, L - Local, X - No Socket
        # Ent --> Number of NHRP entries with same NBMA peer
        NHS Status: E --> Expecting Replies, R --> Responding, W --> Waiting
        UpDn Time --> Up or Down Time for a Tunnel
==========================================================================

Interface: Tunnel3, IPv4 NHRP Details
Type:Hub, NHRP Peers:2,

 # Ent  Peer NBMA Addr Peer Tunnel Add State  UpDn Tm Attrb
 ----- --------------- --------------- ----- -------- -----
     1 96.254.180.226       10.0.252.3    UP 00:31:16     D
     1 15.67.83.114         10.0.252.4    UP 00:30:27     D

R14#sh ip nhrp detail
10.0.252.3/32 via 10.0.252.3
   Tunnel3 created 00:31:44, expire 01:54:32
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 96.254.180.226
10.0.252.4/32 via 10.0.252.4
   Tunnel3 created 00:31:45, expire 01:29:13
   Type: dynamic, Flags: unique registered used nhop
   NBMA address: 15.67.83.114

```

Настроим маршрутизацю iBGP.
R14 и R15.
```
R14#
ip route 10.0.0.0 255.255.0.0 Null0
ip route 172.31.0.0 255.255.0.0 Null0
ip route 192.168.0.0 255.255.0.0 Null0
!
ip prefix-list LOCAL seq 5 permit 10.0.0.0/16
ip prefix-list LOCAL seq 10 permit 172.31.0.0/16
ip prefix-list LOCAL seq 15 permit 192.168.0.0/16
!
route-map SPOKE_ROUTERS permit 10
 match ip address prefix-list LOCAL
!
router bgp 1001
 bgp router-id 10.0.255.14
 bgp log-neighbor-changes
 bgp listen range 10.0.252.0/24 peer-group SPOKES
 neighbor SPOKES peer-group
 neighbor SPOKES remote-as 1001
 !
 address-family ipv4
  network 10.0.255.14 mask 255.255.255.255
  redistribute static route-map SPOKE_ROUTERS
  neighbor SPOKES activate
  neighbor SPOKES route-reflector-client
  neighbor SPOKES route-map SPOKE_ROUTERS out
```
```
R15#
ip route 10.0.0.0 255.255.0.0 Null0
ip route 172.31.0.0 255.255.0.0 Null0
ip route 192.168.0.0 255.255.0.0 Null0
!
ip prefix-list LOCAL seq 5 permit 10.0.0.0/16
ip prefix-list LOCAL seq 10 permit 172.31.0.0/16
ip prefix-list LOCAL seq 15 permit 192.168.0.0/16
!
route-map SPOKE_ROUTERS permit 10
 match ip address prefix-list LOCAL
!
router bgp 1001
 bgp router-id 10.0.255.15
 bgp log-neighbor-changes
 bgp listen range 10.0.252.0/24 peer-group SPOKES
 neighbor SPOKES peer-group
 neighbor SPOKES remote-as 1001
 !
 address-family ipv4
  redistribute static route-map SPOKE_ROUTERS
  neighbor SPOKES activate
  neighbor SPOKES route-reflector-client
  neighbor SPOKES route-map SPOKE_ROUTERS out
```
Настроим SPOKES R27 и R28.
```
R27#
router bgp 1001
 bgp router-id 172.31.255.27
 bgp log-neighbor-changes
 neighbor 10.0.252.1 remote-as 1001
 neighbor 10.0.252.2 remote-as 1001
 !
 address-family ipv4
  network 172.31.255.27 mask 255.255.255.255
  neighbor 10.0.252.1 activate
  neighbor 10.0.252.2 activate
 exit-address-family
```
```
R27#sh ip bgp sum
BGP router identifier 172.31.255.27, local AS number 1001
BGP table version is 92, main routing table version 92
4 network entries using 560 bytes of memory
7 path entries using 560 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1408 total bytes of memory
BGP activity 23/19 prefixes, 45/38 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.252.1      4         1001      55      54       92    0    0 00:45:03        3
10.0.252.2      4         1001      53      56       92    0    0 00:44:51        3

```
```
R28#
router bgp 1001
 bgp router-id 192.168.255.28
 bgp log-neighbor-changes
 neighbor 10.0.252.1 remote-as 1001
 neighbor 10.0.252.2 remote-as 1001
 !
 address-family ipv4
  network 192.168.255.28 mask 255.255.255.255
  neighbor 10.0.252.1 activate
  neighbor 10.0.252.2 activate
 exit-address-family
```
```
R28#sh ip bgp summ
BGP router identifier 192.168.255.28, local AS number 1001
BGP table version is 92, main routing table version 92
4 network entries using 560 bytes of memory
7 path entries using 560 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1408 total bytes of memory
BGP activity 23/19 prefixes, 45/38 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.252.1      4         1001      56      55       92    0    0 00:45:24        3
10.0.252.2      4         1001      56      56       92    0    0 00:45:05        3
```
Проверим связь между SPOKES.
```
R27#ping 192.168.255.28
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.255.28, timeout is 2 seconds:
!!!!!

R27#traceroute  192.168.255.28
Type escape sequence to abort.
Tracing the route to 192.168.255.28
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.252.4 1 msec 0 msec *

```
Связь между SPOKE напрямую, а не через HUB. Отрабатывает протокол NHRP.
```
R27#sh ip ro
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 96.254.180.225 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 96.254.180.225
      10.0.0.0/8 is variably subnetted, 4 subnets, 3 masks
B        10.0.0.0/16 [200/0] via 10.0.252.2, 00:50:16
C        10.0.252.0/24 is directly connected, Tunnel0
L        10.0.252.3/32 is directly connected, Tunnel0
H        10.0.252.4/32 is directly connected, 00:02:40, Tunnel0
      96.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        96.254.180.224/30 is directly connected, Ethernet0/0
L        96.254.180.226/32 is directly connected, Ethernet0/0
      172.31.0.0/16 is variably subnetted, 2 subnets, 2 masks
B        172.31.0.0/16 [200/0] via 10.0.252.2, 00:50:16
C        172.31.255.27/32 is directly connected, Loopback0
B     192.168.0.0/16 [200/0] via 10.0.252.2, 00:50:16
      192.168.255.0/32 is subnetted, 1 subnets
H        192.168.255.28 [250/1] via 10.0.252.4, 00:02:40, Tunnel0

```
