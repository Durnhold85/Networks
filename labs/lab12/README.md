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
!
interface Ethernet0/2
 description R13 to R15
 ip address 10.0.254.14 255.255.255.252
 mpls ip
!
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
R15#
!
mpls label protocol ldp
!
interface Ethernet0/0
 description R15 to R13
 ip address 10.0.254.13 255.255.255.252
 mpls ip
!
interface Ethernet0/1
 description R15 to R12
 ip address 10.0.254.5 255.255.255.252
 mpls ip
!
router ospf 1
 mpls ldp sync
~~~
MPLS работает
~~~
R15#traceroute mpls ipv4 10.0.255.14/32
Tracing MPLS Label Switched Path to 10.0.255.14/32, timeout is 2 seconds

Codes: '!' - success, 'Q' - request not sent, '.' - timeout,
  'L' - labeled output interface, 'B' - unlabeled output interface,
  'D' - DS Map mismatch, 'F' - no FEC mapping, 'f' - FEC mismatch,
  'M' - malformed request, 'm' - unsupported tlvs, 'N' - no label entry,
  'P' - no rx intf label prot, 'p' - premature termination of LSP,
  'R' - transit router, 'I' - unknown upstream index,
  'X' - unknown return code, 'x' - return code 0

Type escape sequence to abort.
  0 10.0.254.5 MRU 1500 [Labels: 29 Exp: 0]
L 1 10.0.254.6 MRU 1504 [Labels: implicit-null Exp: 0] 18 ms
! 2 10.0.254.21 18 ms
~~~
~~~
R14#traceroute mpls ipv4 10.0.255.15/32
Tracing MPLS Label Switched Path to 10.0.255.15/32, timeout is 2 seconds

Codes: '!' - success, 'Q' - request not sent, '.' - timeout,
  'L' - labeled output interface, 'B' - unlabeled output interface,
  'D' - DS Map mismatch, 'F' - no FEC mapping, 'f' - FEC mismatch,
  'M' - malformed request, 'm' - unsupported tlvs, 'N' - no label entry,
  'P' - no rx intf label prot, 'p' - premature termination of LSP,
  'R' - transit router, 'I' - unknown upstream index,
  'X' - unknown return code, 'x' - return code 0

Type escape sequence to abort.
  0 10.0.254.9 MRU 1500 [Labels: 35 Exp: 0]
L 1 10.0.254.10 MRU 1504 [Labels: implicit-null Exp: 0] 14 ms
! 2 10.0.254.13 9 ms
~~~
### Настроим маршрутизаторы в Санкт-Петербурге
Так же уберем суммаризацию с eigrp создадим маршрут по умолчанию и анонсируем его с помощью eigrp.
~~~
R18#
!
mpls label protocol ldp
!
interface Ethernet0/0
 description R18 to R16
 ip address 172.20.254.5 255.255.255.252
 mpls ip
!
interface Ethernet0/1
 description R18 to R17
 ip address 172.20.254.2 255.255.255.252
 mpls ip
!
ip route 0.0.0.0 0.0.0.0 Null0 250
!
ip prefix-list PL-DEFAULT-ONLY seq 5 permit 0.0.0.0/0
!
route-map RM-DEFAULT-ONLY permit 10
 match ip address prefix-list PL-DEFAULT-ONLY
!
router eigrp R18
 !
 address-family ipv4 unicast autonomous-system 1
!
  topology base
   redistribute static route-map RM-DEFAULT-ONLY
~~~
