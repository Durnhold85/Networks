
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
