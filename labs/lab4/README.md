# Настроить протокол IS-IS для IPv4 и IPv6 в ISP Триада.

# План работ:

1. Настроитить IS-IS в ISP Триада.
2. R23 и R25 находятся в зоне 2222.
3. R24 находится в зоне 24.
4. R26 находится в зоне 26.

## Схема стенда.

![](ISIS.png)

### Присвоим NET адреса для процесса isis.

| Equip | Address |
|-------|------|
|  R23  | 49.2222.0101.0025.5023.00 | 
|  R24  | 49.0024.0101.0025.5024.00 |
|  R25  | 49.2222.0101.0025.5025.00 |
|  R26  | 49.0026.0101.0025.5026.00 |

### Включим поддержку ipv6 на всех маршрутизаторах.

```
ipv6 cef

ipv6 unicast-routing
```
Настроим isis на всех маршрутизаторах, включим расширенные метрики metric-style wide, включим multi-topology для поддержки ipv6 протоколом isis, привяжем интерфейсы к isis, сделаем point-to-point линки между маршрутизаторами.
```
R23#
router isis R23
 net 49.2222.0101.0025.5023.00
 is-type level-2-only
 metric-style wide
 !
 address-family ipv6
  multi-topology
 exit-address-family
!
interface Loopback0
 ip address 10.100.255.23 255.255.255.255
 ip router isis R23
 ipv6 enable
 ipv6 router isis R23
!
interface Ethernet0/1
 description R23 to R25
 ip address 10.100.254.5 255.255.255.252
 ip router isis R23
 ipv6 enable
 ipv6 router isis R23
 isis network point-to-point
!
interface Ethernet0/2
 description R23 to R24
 ip address 10.100.254.1 255.255.255.252
 ip router isis R23
 ipv6 enable
 ipv6 router isis R23
 isis network point-to-point
```
```
R24#
router isis R24
 net 49.0024.0101.0025.5024.00
 is-type level-2-only
 metric-style wide
 !
 address-family ipv6
  multi-topology
 exit-address-family
!
interface Loopback0
 ip address 10.100.255.24 255.255.255.255
 ip router isis R24
 ipv6 enable
 ipv6 router isis R24
!
interface Ethernet0/0
 description R24 to R21
 ip address 33.186.35.122 255.255.255.252
!
interface Ethernet0/1
 description R24 to R26
 ip address 10.100.254.9 255.255.255.252
 ip router isis R24
 ipv6 enable
 ipv6 router isis R24
 isis network point-to-point
!
interface Ethernet0/2
 description R24 to R23
 ip address 10.100.254.2 255.255.255.252
 ip router isis R24
 ipv6 enable
 ipv6 router isis R24
 isis network point-to-point
```
```
R25#
router isis R25
 net 49.2222.0101.0025.5025.00
 is-type level-2-only
 metric-style wide
 !
 address-family ipv6
  multi-topology
 exit-address-family
!
interface Loopback0
 ip address 10.100.255.25 255.255.255.255
 ip router isis R25
 ipv6 enable
 ipv6 router isis R25
!
interface Ethernet0/0
 description R25 to R23
 ip address 10.100.254.6 255.255.255.252
 ip router isis R25
 ipv6 enable
 ipv6 router isis R25
 isis network point-to-point
!
interface Ethernet0/2
 description R25 to R26
 ip address 10.100.254.13 255.255.255.252
 ip router isis R25
 ipv6 enable
 ipv6 router isis R25
 isis network point-to-point

```
```
R26#
router isis R26
 net 49.0026.0101.0025.5026.00
 is-type level-2-only
 metric-style wide
 !
 address-family ipv6
  multi-topology
 exit-address-family
!
interface Loopback0
 ip address 10.100.255.26 255.255.255.255
 ip router isis R26
 ipv6 enable
 ipv6 router isis R26
!
interface Ethernet0/0
 description R26 to R24
 ip address 10.100.254.10 255.255.255.252
 ip router isis R26
 ipv6 enable
 ipv6 router isis R26
 isis network point-to-point
!
interface Ethernet0/2
 description R26 to R25
 ip address 10.100.254.14 255.255.255.252
 ip router isis R26
 ipv6 enable
 ipv6 router isis R26
 isis network point-to-point

```
Соседство между маршрутизаторами поднялось.
```
R23#show isis neighbors

Tag R23:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/2       10.100.254.2    UP    23       02
R25            L2   Et0/1       10.100.254.6    UP    24       01
```
```
R24#sh isis neighbors

Tag R24:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R23            L2   Et0/2       10.100.254.1    UP    22       02
R26            L2   Et0/1       10.100.254.10   UP    29       01
```
```
R25#sh isis neighbors

Tag R25:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R23            L2   Et0/0       10.100.254.5    UP    22       01
R26            L2   Et0/2       10.100.254.14   UP    26       02
```
```
R26#show isis neighbors

Tag R26:
System Id      Type Interface   IP Address      State Holdtime Circuit Id
R24            L2   Et0/0       10.100.254.9    UP    27       01
R25            L2   Et0/2       10.100.254.13   UP    29       02
```
