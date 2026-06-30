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
