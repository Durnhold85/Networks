# Настроить BGP free core в офисах Москвы и Санкт-Петербурга.

# План работ:

1. Настроить BGP free core в офисе Москвы.
2. Настроить BGP free core в офисе Санкт-Петербурга.

### Настроим маршрутизаторы в Москве

~~~
R14#
!
mpls label protocol ldp
interface Ethernet0/0
 ip address 10.0.254.21 255.255.255.252
 mpls ip
!
interface Ethernet0/1
 ip address 10.0.254.9 255.255.255.252
 mpls ip
!
router ospf 1
 mpls ldp sync
 router-id 10.0.255.14
~~~

~~~
R12#
!
mpls label protocol ldp
interface Ethernet0/2
 description R12 to R14
 ip address 10.0.254.22 255.255.255.252
 mpls ip
interface Ethernet0/3
 description R12 to R15
 ip address 10.0.254.6 255.255.255.252
 mpls ip
!
router ospf 1
 mpls ldp sync
 router-id 10.0.255.12
~~~

~~~
R13#
!
mpls label protocol ldp
!interface Ethernet0/2
 description R13 to R15
 ip address 10.0.254.14 255.255.255.252
 mpls ip

interface Ethernet0/3
 description R13 to R14
 ip address 10.0.254.10 255.255.255.252
 mpls ip
!
router ospf 1
 mpls ldp sync
 router-id 10.0.255.13
~~~

~~~

~~~
