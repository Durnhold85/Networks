
### OSPF. Фильтрация

### План работ:

1. Маршрутизаторы R14-R15 находятся в зоне 0 - backbone.
2. Маршрутизаторы R12-R13 находятся в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию.
3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию.
4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101.

Схема.

![](OSPF.png)

Настроим зону AREA0 (Backbone).

```
R14#
router ospf 1
 router-id 10.0.255.14
!
interface Ethernet0/0
 ip address 10.0.254.21 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/1
 ip address 10.0.254.9 255.255.255.252
 ip ospf 1 area 0
```
```
R15#
router ospf 1
 router-id 10.0.255.15
!
interface Ethernet0/0
 description R15 to R13
 ip address 10.0.254.13 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/1
 description R15 to R12
 ip address 10.0.254.5 255.255.255.252
 ip ospf 1 area 0
```

```
R12#
router ospf 1
 router-id 10.0.255.12
!
interface Ethernet0/2
 description R12 to R14
 ip address 10.0.254.22 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/3
 description R12 to R15
 ip address 10.0.254.6 255.255.255.252
 ip ospf 1 area 0
```
```
R13#
router ospf 1
 router-id 10.0.255.13
!
interface Ethernet0/2
 description R13 to R15
 ip address 10.0.254.14 255.255.255.252
 ip ospf 1 area 0
!
interface Ethernet0/3
 description R13 to R14
 ip address 10.0.254.10 255.255.255.252
 ip ospf 1 area 0
```

Соседство между маршрутизаторами поднялось.

```
R12#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.14       1   FULL/DR         00:00:38    10.0.254.21     Ethernet0/2
10.0.255.15       1   FULL/DR         00:00:37    10.0.254.5      Ethernet0/3
```
```
R13#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.15       1   FULL/DR         00:00:35    10.0.254.13     Ethernet0/2
10.0.255.14       1   FULL/DR         00:00:33    10.0.254.9      Ethernet0/3
```
```
R14#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.12       1   FULL/BDR        00:00:32    10.0.254.22     Ethernet0/0
10.0.255.13       1   FULL/BDR        00:00:31    10.0.254.10     Ethernet0/1
```
```
R15#sh ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.255.13       1   FULL/BDR        00:00:31    10.0.254.14     Ethernet0/0
10.0.255.12       1   FULL/BDR        00:00:33    10.0.254.6      Ethernet0/1
```
Настроим AREA 10
```
R12#
interface Ethernet0/0
 description R12 to SW4
 ip address 10.0.254.37 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/1
 description R12 to SW5
 ip address 10.0.254.29 255.255.255.252
 ip ospf 1 area 10
```
```
R13#
interface Ethernet0/0
 description R13 to SW5
 ip address 10.0.254.33 255.255.255.252
 ip ospf 1 area 10
!
interface Ethernet0/1
 description R13 to SW4
 ip address 10.0.254.25 255.255.255.252
 ip ospf 1 area 10
```
