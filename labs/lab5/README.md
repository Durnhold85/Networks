# Настроить EIGRP named-mode для IPv4 и IPv6 в офисе С.-Петербург.

# План работ:

1. В офисе С.-Петербург настроить EIGRP.
2. R32 получает только маршрут по умолчанию.
3. R16-17 анонсируют только суммарные префиксы.
4. Использовать EIGRP named-mode для настройки сети.

Настройка осуществляется одновременно для IPv4 и IPv6.

## Схема стенда.

![](EIGRP.png)

### Включим поддержку ipv6 на всех маршрутизаторах.

```
ipv6 cef
ipv6 unicast-routing
```
### Настроим EIGRP , включим поддержку ipv6, так же включим ipv6 на интерфейсах.

На R16 настроим суммарный маршрут по умолчанию 0.0.0.0 0.0.0.0 в сторону R32, в сторону SW9 и SW10 настроим анонс суммарного маршрута 172.20.254.0 255.255.254.0 
```
R16#
router eigrp R16
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   summary-address 0.0.0.0 0.0.0.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   summary-address 172.20.254.0 255.255.254.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   summary-address 172.20.254.0 255.255.254.0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.254.0 0.0.1.255
  eigrp router-id 172.20.255.16
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!
interface Ethernet0/0
 description R16 to SW10
 ip address 172.20.254.9 255.255.255.252
 ipv6 enable
!
interface Ethernet0/1
 description R16 to R18
 ip address 172.20.254.6 255.255.255.252
 ipv6 enable
!
interface Ethernet0/2
 description R16 to SW9
 ip address 172.20.254.21 255.255.255.252
 ipv6 enable
!
interface Ethernet0/3
 description R16 to R32
 ip address 172.20.254.25 255.255.255.252
 ipv6 enable
```

На R17 пропишем анонс суммарного маршута в сторону SW9 и SW10 172.20.254.0 255.255.254.0
```
R17#
router eigrp R17
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   summary-address 172.20.254.0 255.255.254.0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   summary-address 172.20.254.0 255.255.254.0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.254.0 0.0.1.255
  eigrp router-id 172.20.255.17
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/2
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!
interface Ethernet0/0
 description R17 to SW9
 ip address 172.20.254.13 255.255.255.252
 ipv6 enable
!
interface Ethernet0/1
 description R17 to R18
 ip address 172.20.254.1 255.255.255.252
 ipv6 enable
!
interface Ethernet0/2
 description R17 to SW10
 ip address 172.20.254.17 255.255.255.252
 ipv6 enable
```

Настроим R18,R32,SW9 и SW10

```
R18#
router eigrp R18
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.254.0 0.0.1.255
  eigrp router-id 172.20.255.18
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/1
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!
interface Ethernet0/0
 description R18 to R16
 ip address 172.20.254.5 255.255.255.252
 ipv6 enable
!
interface Ethernet0/1
 description R18 to R17
 ip address 172.20.254.2 255.255.255.252
 ipv6 enable
```
```
R32#
router eigrp R32
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.254.0 0.0.1.255
  eigrp router-id 172.20.255.32
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!
interface Ethernet0/0
 description R32 to SW16
 ip address 172.20.254.26 255.255.255.252
 ipv6 enable
```
```
SW9#
router eigrp SW9
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet1/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.0.0
  eigrp router-id 172.20.255.9
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet1/0
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
interface Ethernet0/3
 description SW9 to R17
 no switchport
 ip address 172.20.254.14 255.255.255.252
 duplex auto
 ipv6 enable
!
interface Ethernet1/0
 description SW9 to R16
 no switchport
 ip address 172.20.254.22 255.255.255.252
 duplex auto
 ipv6 enable
```
```
SW10#
router eigrp SW10
 !
 address-family ipv4 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet1/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
  network 172.20.0.0
  eigrp router-id 172.20.255.10
 exit-address-family
 !
 address-family ipv6 unicast autonomous-system 1
  !
  af-interface default
   passive-interface
  exit-af-interface
  !
  af-interface Ethernet0/3
   no passive-interface
  exit-af-interface
  !
  af-interface Ethernet1/0
   no passive-interface
  exit-af-interface
  !
  topology base
  exit-af-topology
 exit-address-family
!
interface Ethernet0/3
 description SW10 to R16
 no switchport
 ip address 172.20.254.10 255.255.255.252
 duplex auto
 ipv6 enable
!
interface Ethernet1/0
 description SW10 to R17
 no switchport
 ip address 172.20.254.18 255.255.255.252
 duplex auto
 ipv6 enable
```
На SW9 и SW10 пришёл суммарный маршрут 172.20.254.0/23

```
SW9#sh ip ro eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      172.20.0.0/16 is variably subnetted, 10 subnets, 4 masks
D        172.20.254.0/23 [90/1024640] via 172.20.254.21, 02:02:48, Ethernet1/0
                         [90/1024640] via 172.20.254.13, 02:02:48, Ethernet0/3
SW9#

```

```
SW10#sh ip ro eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      172.20.0.0/16 is variably subnetted, 10 subnets, 4 masks
D        172.20.254.0/23 [90/1024640] via 172.20.254.17, 02:02:02, Ethernet1/0
                         [90/1024640] via 172.20.254.9, 02:02:02, Ethernet0/3

```
На R32 приходит маршрут по умолчанию, другие маршруты в eigrp отсутствуют.
```
R32#sh ip ro eigrp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 172.20.254.25 to network 0.0.0.0

D*    0.0.0.0/0 [90/1024640] via 172.20.254.25, 02:03:44, Ethernet0/0
```
