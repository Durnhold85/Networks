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
