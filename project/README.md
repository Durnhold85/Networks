# Организация сети в центральном офисе и филиалах.

## Цель проектной работы организовать выход в интернет в главном офисе и филиалах, а так же связать офис и филиалы между собой.

### План работ:

1. Разработать и задокументировать адресное пространство для лабораторного стенда.
2. Настроить локальную сеть в офисе HQ. Протокол маршрутизации OSPF, HSRP, DHCP, RSTP, VLAN.
3. Настроить связность между провайдерами и офисом HQ по протоколу BGP.
4. Настроить NAT в офисе HQ и филиалах.
6. Настроить связность между офисом HQ и филиалами. (DMVPN, IPSEC, IBGP).

#### Схема

![](general_scheme.png)

#### Префиксы по регионам:
| Регион | ipv4 |
|-------|------|
|HQ |10.0.0.0/16|
|office1|172.20.0.0/16|
|office2|172.31.0.0/16|
|office3|192.168.0.0/16|

### IP адреса соединений между регионами:
| Equip | ipv4 | 
|-------|------|
| HQ(R1) - ISP1| 45.68.214.4/30|
| HQ(R2) - ISP4 | 12.136.47.88/30|
| ISP1 - ISP2 | 189.45.13.28/30|
| ISP2 - ISP3 | 156.78.22.36/30|
| ISP2 - ISP4 | 74.65.10.12/30|
| ISP2 - ISP5 | 67.167.89.32/30|
| ISP2 - office3 | 23.156.78.32/30|
| ISP3 - ISP5 | 153.68.28.128/30|
| ISP3 - office2 | 99.15.28.232/30|
| ISP4 - ISP5 | 138.18.35.212/30|
| ISP5 - office1 | 132.185.67.56/30|

### ISP1 AS568

| Equip | Port | Address                  | Network                | Description   |
|-------|------|--------------------------|------------------------|---------------|
|  ISP1  | e0/0 | 45.68.214.6/30          |45.68.214.4/30          | ISP1 to HQ(R1)|
|  ISP1  | e0/1 | 189.45.13.30/30         |189.45.13.28/30         | ISP1 to ISP2  |

### ISP2 AS329

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
| ISP2  | e0/0 | 189.45.13.29/30          |189.45.13.28/30         | ISP2 to ISP1   |
| ISP2  | e0/1 | 67.167.89.33/30          |67.167.89.32/30         | ISP2 to ISP5   |
| ISP2  | e0/2 | 156.78.22.37/30          |156.78.22.36/30         | ISP2 to ISP3   |
| ISP2  | e0/3 | 74.65.10.14/30           |74.65.10.12/30          | ISP2 to ISP4   |
| ISP2  | e1/0 | 23.156.78.33/30          |23.156.78.32/30         | ISP2 to office3|

### ISP3 AS492

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
| ISP3  | e0/0 | 156.78.22.38/30          |156.78.22.36/30         | ISP3 to ISP2   |
| ISP3  | e0/1 | 153.68.28.130/30         |153.68.28.128/30        | ISP3 to ISP5   |
| ISP3  | e0/2 | 99.15.28.233/30          |99.15.28.232/30         | ISP3 to office2|

### ISP4 AS64606

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
| ISP4  | e0/0 | 12.136.47.89/30          |12.136.47.88/30         | ISP4 to HQ(R2) |
| ISP4  | e0/1 | 138.18.35.213/30         |138.18.35.212/30        | ISP4 to ISP5   |
| ISP4  | e0/2 | 74.65.10.13/30           |74.65.10.12/30          | ISP4 to ISP2   | 

### ISP5 AS3892

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
| ISP5  | e0/0 | 138.18.35.214/30         |138.18.35.212/30        | ISP5 to ISP4   |
| ISP5  | e0/1 | 67.167.89.34/30          |67.167.89.32/30         | ISP5 to ISP2   |
| ISP5  | e0/2 | 132.185.67.57/30         |132.185.67.56/30        | ISP2 to office1|
| ISP5  | e0/3 | 153.68.28.129/30         |153.68.28.128/30        | ISP5 to ISP3   |

### HQ AS1249

| Equip | Port | Address                  | Network                | Description   |
|-------|------|--------------------------|------------------------|---------------|
|  R1   | e0/0        | 45.68.214.5/30          |45.68.214.4/30         | R1 to ISP1    |
|  R1   | e0/1        | 10.0.254.5/30           |10.0.254.4/30          | R1 to SW4     |
|  R1   | e0/2        | 10.0.254.1/30           |10.0.254.0/30          | R1 to SW3     |
|  R1   | Lo0         | 10.0.255.10/32          |                       | Loopback      |
|  R2   | e0/0        | 12.136.47.90/30         |12.136.47.88/30        | R2 to ISP4    |
|  R2   | e0/1        | 10.0.254.9/30           |10.0.254.8/30          | R2 to SW3     |
|  R2   | e0/2        | 10.0.254.13/30          |10.0.254.12/30         | R2 to SW4     |
|  R2   | Lo0         | 10.0.255.20/32          |                       | Loopback      |
|  SW3  | e0/0        | 10.0.254.10/30          |10.0.254.8/30          | SW3 to R2     |
|  SW3  | e0/1        | 10.0.254.2/30           |10.0.254.0/30          | SW3 to R1     |
|  SW3  | int Vl30    | 10.0.30.1/24            |10.0.30.0/24           | int vlan 30   |
|  SW3  | Lo0         | 10.0.255.30/32          |                       | Loopback      |
|  SW4  | e0/0        | 10.0.254.6/30           |10.0.254.4/30          | SW4 to R1     |
|  SW4  | e0/1        | 10.0.254.14/30          |10.0.254.12/30         | SW4 to R2     |
|  SW4  | int Vl40    | 10.0.40.1/24            |10.0.40.0/24           | int Vlan 40   |
|  SW4  | Lo0         | 10.0.255.40/32          |                       | Loopback      |
|  SW5  | mgmt ip     | 10.0.30.254/24          |10.0.30.0/24           | Management    |
|  SW6  | mgmt ip     | 10.0.40.254/24          |10.0.40.0/24           | Management    |

Схема HQ.

![](HQ.png)

### Office1

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
|Office1| e0/0 | 132.185.67.58/30         |132.185.67.56/30        |Office1 to ISP5 |
|Office1| e0/1 | 172.20.10.1/2            |172.20.10.0/24          | Gateway        |
|Office1| lo0  | 172.20.255.10            |                        | Loopback       |
|Office1| LAN  |                          |172.20.10.0/24          | LAN            |

### Office2

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
|Office2| e0/0 | 99.15.28.234/30          |99.15.28.232/30         |Office2 to ISP3 |
|Office2| e0/1 | 172.31.20.1/24           |172.31.20.0/24          | Gateway        |
|Office2| lo0  | 172.31.255.20            |                        | Loopback       |
|Office2| LAN  |                          |172.31.20.0/24          | LAN            |
### Office3

| Equip | Port | Address                  | Network                | Description    |
|-------|------|--------------------------|------------------------|----------------|
|Office3| e0/0 | 23.156.78.34/30          |23.156.78.32/30         |Office3 to ISP2 |
|Office3| lo0  | 192.168.30.1/24          |192.168.30.0/24         | Gateway        |
|Office3| lo0  | 192.168.255.30           |                        | Loopback       |
|Office3| lo0  |                          |192.168.30.0/24         | LAN            |

### Настроим L2 сегмент в офисе HQ.(HSRP,RSTP,Vlan)
~~~
SW3#
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 1,30 priority 4096
spanning-tree vlan 40 priority 16384
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Vlan30
 ip address 10.0.30.2 255.255.255.0
 standby 30 ip 10.0.30.1
 standby 30 priority 150
 standby 30 preempt
!
interface Vlan40
 ip address 10.0.40.2 255.255.255.0
 standby 40 ip 10.0.40.1
 standby 40 priority 110
 standby 40 preempt
~~~
~~~
SW4#
spanning-tree mode rapid-pvst
spanning-tree extend system-id
spanning-tree vlan 1,30 priority 16384
spanning-tree vlan 40 priority 4096
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Vlan30
 ip address 10.0.30.3 255.255.255.0
 standby 30 ip 10.0.30.1
 standby 30 priority 110
 standby 30 preempt
!
interface Vlan40
 ip address 10.0.40.3 255.255.255.0
 standby 40 ip 10.0.40.1
 standby 40 priority 150
 standby 40 preempt
~~~
~~~
SW3#sh standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl30        30   150 P Active  local           10.0.30.3       10.0.30.1
Vl40        40   110 P Standby 10.0.40.3       local           10.0.40.1
~~~
~~~
SW4#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl30        30   110 P Standby 10.0.30.2       local           10.0.30.1
Vl40        40   150 P Active  local           10.0.40.2       10.0.40.1
~~~
### Настроим DHCP server на SW3.
~~~
SW3#
ip dhcp excluded-address 10.0.30.1 10.0.30.3
ip dhcp excluded-address 10.0.40.1 10.0.40.3
!
ip dhcp pool Vlan30
 network 10.0.30.0 255.255.255.0
 default-router 10.0.30.1
!
ip dhcp pool Vlan40
 network 10.0.40.0 255.255.255.0
 default-router 10.0.40.1
~~~
### Настроим OSPF в офисе HQ.
~~~
R1#
router ospf 1
 router-id 10.0.255.10
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 default-information originate metric 20
!
interface Loopback0
 ip address 10.0.255.10 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_SW4
 ip address 10.0.254.5 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_SW3
 ip address 10.0.254.1 255.255.255.252
 ip ospf 1 area 0
~~~
~~~
R2#
interface Loopback0
 ip address 10.0.255.20 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/1
 description to_SW3
 ip address 10.0.254.9 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_SW4
 ip address 10.0.254.13 255.255.255.252
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.0.255.20
 passive-interface default
 no passive-interface Ethernet0/1
 no passive-interface Ethernet0/2
 default-information originate metric 10
~~~
~~~
SW3#
interface Loopback0
 ip address 10.0.255.30 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_R2
 no switchport
 ip address 10.0.254.10 255.255.255.252
 ip ospf 1 area 0
 duplex auto
!
interface Ethernet0/1
 description to_R1
 no switchport
 ip address 10.0.254.2 255.255.255.252
 ip ospf 1 area 0
 duplex auto
!
interface Vlan30
 ip address 10.0.30.2 255.255.255.0
 standby 30 ip 10.0.30.1
 standby 30 priority 150
 standby 30 preempt
 ip ospf 1 area 0
!
interface Vlan40
 ip address 10.0.40.2 255.255.255.0
 standby 40 ip 10.0.40.1
 standby 40 priority 110
 standby 40 preempt
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.0.255.30
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
~~~
~~~
SW4#
interface Loopback0
 ip address 10.0.255.40 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
!
interface Ethernet0/0
 description to_R1
 no switchport
 ip address 10.0.254.6 255.255.255.252
 ip ospf 1 area 0
 duplex auto
!
interface Ethernet0/1
 description to_R2
 no switchport
 ip address 10.0.254.14 255.255.255.252
 ip ospf 1 area 0
 duplex auto
!
interface Vlan30
 ip address 10.0.30.3 255.255.255.0
 standby 30 ip 10.0.30.1
 standby 30 priority 110
 standby 30 preempt
 ip ospf 1 area 0
!
interface Vlan40
 ip address 10.0.40.3 255.255.255.0
 standby 40 ip 10.0.40.1
 standby 40 priority 150
 standby 40 preempt
 ip ospf 1 area 0
!
router ospf 1
 router-id 10.0.255.40
 passive-interface default
 no passive-interface Ethernet0/0
 no passive-interface Ethernet0/1
~~~
Соседство поднялось.
~~~
R1#sh ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.30       1   FULL/DR         00:00:39    10.0.254.2      Ethernet0/2
10.0.255.40       1   FULL/BDR        00:00:36    10.0.254.6      Ethernet0/1
~~~
~~~
R2#sh ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.40       1   FULL/BDR        00:00:33    10.0.254.14     Ethernet0/2
10.0.255.30       1   FULL/DR         00:00:37    10.0.254.10     Ethernet0/1
~~~
~~~
SW3#sh ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.10       1   FULL/BDR        00:00:39    10.0.254.1      Ethernet0/1
10.0.255.20       1   FULL/BDR        00:00:37    10.0.254.9      Ethernet0/0
~~~
~~~
SW4#sh ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.10       1   FULL/DR         00:00:32    10.0.254.5      Ethernet0/0
10.0.255.20       1   FULL/DR         00:00:36    10.0.254.13     Ethernet0/1
~~~
### Настроим связность между провайдерами, а так же настроим подключение офиса HQ к провайдерам по BGP.
От провайдера ISP1 в сторону HQ(R1) настроим только дефолт.
~~~
ISP1#
ip prefix-list DEFAULT seq 10 permit 0.0.0.0/0
!
route-map ADVERTISE-ONLY-DEFAULT permit 10
 match ip address prefix-list DEFAULT
!
router bgp 568
 bgp log-neighbor-changes
 network 45.68.214.4 mask 255.255.255.252
 neighbor 45.68.214.5 remote-as 1249
 neighbor 45.68.214.5 default-originate
 neighbor 45.68.214.5 route-map ADVERTISE-ONLY-DEFAULT out
 neighbor 189.45.13.29 remote-as 329
~~~
От провайдера ISP4 в сторону HQ(R2) также анонсируем только дефолт.
~~~
ISP4#
ip prefix-list DEFAULT seq 10 permit 0.0.0.0/0
!
route-map ADVERTISE-ONLY-DEFAULT permit 10
 match ip address prefix-list DEFAULT
!
router bgp 64606
 bgp log-neighbor-changes
 network 12.136.47.88 mask 255.255.255.252
 neighbor 12.136.47.90 remote-as 1249
 neighbor 12.136.47.90 default-originate
 neighbor 12.136.47.90 route-map ADVERTISE-ONLY-DEFAULT out
 neighbor 74.65.10.14 remote-as 329
 neighbor 138.18.35.214 remote-as 3892
~~~
В офисе HQ настроим связность с провайдерами ISP1 и ISP4, ISP4 сделаем более предпочтительным.
Уберем транзит через нашу AS.
~~~
R1#
ip as-path access-list 1 permit ^$
!
route-map PREPEND-ISP1 permit 10
 set as-path prepend 1249 1249
!
router bgp 1249
 bgp log-neighbor-changes
 neighbor 10.0.255.20 remote-as 1249
 neighbor 10.0.255.20 update-source Loopback0
 neighbor 45.68.214.6 remote-as 568
 !
 address-family ipv4
  neighbor 10.0.255.20 activate
  neighbor 10.0.255.20 next-hop-self
  neighbor 45.68.214.6 activate
  neighbor 45.68.214.6 route-map PREPEND-ISP1 out
  neighbor 45.68.214.6 filter-list 1 out
 exit-address-family
~~~
~~~
R2#
ip as-path access-list 1 permit ^$
!
route-map PREFER-ISP4 permit 10
 set local-preference 150
!
router bgp 1249
 bgp log-neighbor-changes
 neighbor 10.0.255.10 remote-as 1249
 neighbor 10.0.255.10 update-source Loopback0
 neighbor 12.136.47.89 remote-as 64606
 !
 address-family ipv4
  neighbor 10.0.255.10 activate
  neighbor 10.0.255.10 next-hop-self
  neighbor 12.136.47.89 activate
  neighbor 12.136.47.89 route-map PREFER-ISP4 in
  neighbor 12.136.47.89 filter-list 1 out
 exit-address-family
~~~
Связность с ISP4 и ISP1 поднялась. 
~~~
R1#sh ip bgp sum
BGP router identifier 10.0.255.10, local AS number 1249
BGP table version is 28, main routing table version 28
1 network entries using 140 bytes of memory
2 path entries using 160 bytes of memory
2/1 BGP path/bestpath attribute entries using 288 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 636 total bytes of memory
BGP activity 18/17 prefixes, 46/44 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.255.20     4         1249      55      57       28    0    0 00:43:44        1
45.68.214.6     4          568     153     132       28    0    0 01:51:21        1
~~~
~~~
R2(config)#do sh ip bgp sum
BGP router identifier 10.0.255.20, local AS number 1249
BGP table version is 2, main routing table version 2
1 network entries using 140 bytes of memory
1 path entries using 80 bytes of memory
2/1 BGP path/bestpath attribute entries using 288 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 532 total bytes of memory
BGP activity 13/12 prefixes, 42/41 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.255.10     4         1249      56      55        2    0    0 00:43:00        0
12.136.47.89    4        64606      52      51        2    0    0 00:43:00        1
~~~
Настроим связность между остальными ISP.
~~~
ISP2#
router bgp 329
 bgp log-neighbor-changes
 network 23.156.78.32 mask 255.255.255.252
 neighbor 67.167.89.34 remote-as 3892
 neighbor 74.65.10.13 remote-as 64606
 neighbor 156.78.22.38 remote-as 492
 neighbor 189.45.13.30 remote-as 568
~~~
~~~
ISP3#
router bgp 492
 bgp log-neighbor-changes
 network 99.15.28.232 mask 255.255.255.252
 neighbor 153.68.28.129 remote-as 3892
 neighbor 156.78.22.37 remote-as 329
~~~
~~~
ISP5#
router bgp 3892
 bgp log-neighbor-changes
 network 132.185.67.56 mask 255.255.255.252
 neighbor 67.167.89.33 remote-as 329
 neighbor 138.18.35.213 remote-as 64606
 neighbor 153.68.28.130 remote-as 492
~~~
~~~
ISP2#sh ip bgp summ
BGP router identifier 189.45.13.29, local AS number 329
BGP table version is 13, main routing table version 13
5 network entries using 700 bytes of memory
10 path entries using 800 bytes of memory
10/5 BGP path/bestpath attribute entries using 1440 bytes of memory
9 BGP AS-PATH entries using 216 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3156 total bytes of memory
BGP activity 8/3 prefixes, 15/5 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
67.167.89.34    4         3892     217     215       13    0    0 03:05:29        3
74.65.10.13     4        64606     245     252       13    0    0 03:37:35        2
156.78.22.38    4          492     268     271       13    0    0 03:54:08        3
189.45.13.30    4          568     285     289       13    0    0 04:13:11        1
~~~
~~~
ISP3#sh ip bgp sum
BGP router identifier 156.78.22.38, local AS number 492
BGP table version is 16, main routing table version 16
5 network entries using 700 bytes of memory
9 path entries using 720 bytes of memory
9/5 BGP path/bestpath attribute entries using 1296 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2908 total bytes of memory
BGP activity 8/3 prefixes, 15/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
153.68.28.129   4         3892     219     215       16    0    0 03:05:32        4
156.78.22.37    4          329     271     269       16    0    0 03:54:44        4
~~~
~~~
ISP5#sh ip bgp sum
BGP router identifier 153.68.28.129, local AS number 3892
BGP table version is 17, main routing table version 17
5 network entries using 700 bytes of memory
12 path entries using 960 bytes of memory
12/5 BGP path/bestpath attribute entries using 1728 bytes of memory
11 BGP AS-PATH entries using 264 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3652 total bytes of memory
BGP activity 8/3 prefixes, 17/5 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
67.167.89.33    4          329     215     216       17    0    0 03:04:52        4
138.18.35.213   4        64606     214     215       17    0    0 03:05:27        4
153.68.28.130   4          492     214     218       17    0    0 03:04:19        3
~~~
### Настроим NAT в офисе HQ.
~~~
R1#
access-list 1 permit 10.0.0.0 0.0.255.255
!
ip nat inside source list 1 interface Ethernet0/0 overload
!
interface Ethernet0/0
 description to_ISP1
 ip address 45.68.214.5 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/1
 description to_SW4
 ip address 10.0.254.5 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_SW3
 ip address 10.0.254.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
~~~
~~~
R2#
access-list 1 permit 10.0.0.0 0.0.255.255
!
ip nat inside source list 1 interface Ethernet0/0 overload
!
interface Ethernet0/0
 description to_ISP4
 ip address 12.136.47.90 255.255.255.252
 ip nat outside
 ip virtual-reassembly in
!
interface Ethernet0/1
 description to_SW3
 ip address 10.0.254.9 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
!
interface Ethernet0/2
 description to_SW4
 ip address 10.0.254.13 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
~~~
~~~
R2(config-if)#do sh ip nat tra
Pro Inside global      Inside local       Outside local      Outside global
icmp 12.136.47.90:26022 10.0.40.4:26022   99.15.28.233:26022 99.15.28.233:26022
icmp 12.136.47.90:26278 10.0.40.4:26278   99.15.28.233:26278 99.15.28.233:26278
icmp 12.136.47.90:26534 10.0.40.4:26534   99.15.28.233:26534 99.15.28.233:26534
icmp 12.136.47.90:26790 10.0.40.4:26790   99.15.28.233:26790 99.15.28.233:26790
icmp 12.136.47.90:27046 10.0.40.4:27046   99.15.28.233:27046 99.15.28.233:27046
~~~
