# Настроить eBGP/iBGP IPv6 unicast для всех сегментов сети по аналогичной логике с настройкой eBGP/iBGP IPv4 unicast.

# План работ:

1. Настроить eBGP IPv6 unicast между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроить eBGP IPv6 unicast между провайдерами Киторн и Ламас.
3. Настроить eBGP IPv6 unicast между Ламас и Триада.
4. Настроить eBGP IPv6 unicast между офисом С.-Петербург и провайдером Триада.
5. Настроить IPv6 unicast связность между пограничными роутерами офисов Москва и С.-Петербург.
6. Настроить iBGP IPv6 unicast в офисе Москва между маршрутизаторами R14 и R15.
7. Настроить iBGP IPv6 unicast в провайдере Триада, с использованием RR.
8. Все офисы в лабораторной работе должны иметь IP связность.

### Настроим eBGP между Москвой, ISP Киторн и ISP Ламас, между ISP Киторн и ISP Ламас, а так же настроим iBGP между R14 и R15 в Москве.
Москва.
~~~
R14#
router bgp 1001
 bgp router-id 10.0.255.14
 bgp log-neighbor-changes
 neighbor 2001:DB8:1:14::2 remote-as 101
 neighbor 2001:DB8:10::15 remote-as 1001
 neighbor 2001:DB8:10::15 update-source Loopback0
 !
 address-family ipv6
  network 2001:DB8:10::14/128
  neighbor 2001:DB8:1:14::2 activate
  neighbor 2001:DB8:10::15 activate
  neighbor 2001:DB8:10::15 next-hop-self
 exit-address-family
~~~
~~~
R15#
router bgp 1001
 bgp router-id 10.0.255.15
 bgp log-neighbor-changes
 neighbor 2001:DB8:2:15::2 remote-as 301
 neighbor 2001:DB8:10::14 remote-as 1001
 neighbor 2001:DB8:10::14 update-source Loopback0
 !
 address-family ipv6
  network 2001:DB8:10::15/128
  neighbor 2001:DB8:2:15::2 activate
  neighbor 2001:DB8:10::14 activate
  neighbor 2001:DB8:10::14 next-hop-self
 exit-address-family
~~~
ISP Киторн
~~~
R22#
router bgp 101
 bgp router-id 193.12.255.22
 bgp log-neighbor-changes
 neighbor 2001:DB8:1:14::1 remote-as 1001
 neighbor 2001:DB8:2:12::2 remote-as 301
 neighbor 2001:DB8:2:23::2 remote-as 520
 !
 address-family ipv6
  neighbor 2001:DB8:1:14::1 activate
  neighbor 2001:DB8:2:12::2 activate
  neighbor 2001:DB8:2:23::2 activate
 exit-address-family
~~~
ISP Ламас
~~~
R21#
router bgp 301
 bgp router-id 37.24.255.21
 bgp log-neighbor-changes
 neighbor 2001:DB8:2:12::1 remote-as 101
 neighbor 2001:DB8:2:15::1 remote-as 1001
 neighbor 2001:DB8:2:34::2 remote-as 520
 !
 address-family ipv6
  neighbor 2001:DB8:2:12::1 activate
  neighbor 2001:DB8:2:15::1 activate
  neighbor 2001:DB8:2:34::2 activate
 exit-address-family
~~~
Проверим, соседство по BGP ipv6 поднялось.
~~~
R14#show bgp ipv6 unicast summary
BGP router identifier 10.0.255.14, local AS number 1001
BGP table version is 5, main routing table version 5
3 network entries using 492 bytes of memory
4 path entries using 416 bytes of memory
4/3 BGP path/bestpath attribute entries using 576 bytes of memory
4 BGP AS-PATH entries using 112 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1596 total bytes of memory
BGP activity 32/0 prefixes, 38/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:1:14::2
                4          101     527     523        5    0    0 07:53:08        1
2001:DB8:10::15 4         1001     521     524        5    0    0 07:52:11        2
~~~
~~~
R15#sh bgp ipv6 unicast summary
BGP router identifier 10.0.255.15, local AS number 1001
BGP table version is 4, main routing table version 4
3 network entries using 492 bytes of memory
3 path entries using 312 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
2 BGP rrinfo entries using 48 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1332 total bytes of memory
BGP activity 32/0 prefixes, 40/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:2:15::2
                4          301     498     494        4    0    0 07:27:29        1
2001:DB8:10::14 4         1001     525     523        4    0    0 07:53:10        1
~~~
~~~
R22(config)#do sh bgp ipv6 unicast summary
BGP router identifier 193.12.255.22, local AS number 101
BGP table version is 4, main routing table version 4
3 network entries using 492 bytes of memory
6 path entries using 624 bytes of memory
5/3 BGP path/bestpath attribute entries using 720 bytes of memory
11 BGP AS-PATH entries using 296 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2132 total bytes of memory
BGP activity 82/41 prefixes, 214/107 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:1:14::1
                4         1001       9       7        4    0    0 00:01:58        3
2001:DB8:2:12::2
                4          301      10       8        4    0    0 00:01:58        3
2001:DB8:2:23::2
                4          520       6       8        4    0    0 00:01:58        0
~~~
~~~
R21#sh bgp ipv6 unicast summary
BGP router identifier 37.24.255.21, local AS number 301
BGP table version is 4, main routing table version 4
3 network entries using 492 bytes of memory
5 path entries using 520 bytes of memory
4/3 BGP path/bestpath attribute entries using 576 bytes of memory
8 BGP AS-PATH entries using 192 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1780 total bytes of memory
BGP activity 41/0 prefixes, 57/2 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:2:12::1
                4          101     217     218        4    0    0 03:13:05        2
2001:DB8:2:15::1
                4         1001     492     496        4    0    0 07:25:52        2
2001:DB8:2:34::2
                4          520     115     115        4    0    0 01:41:07        1
~~~
### Настроим iBGP в ISP Триада и eBGP с ISP Киторн, ISP Ламас и Санкт-Петербург.
~~~
R23#sh run | sec bgp
router bgp 520
 bgp router-id 10.100.255.23
 bgp log-neighbor-changes
 neighbor 2001:DB8:2:23::1 remote-as 101
 neighbor 2001:DB8:3:25::1 remote-as 520
 neighbor 2001:DB8:3:25::1 update-source Loopback0
 !
 address-family ipv6
  neighbor 2001:DB8:2:23::1 activate
  neighbor 2001:DB8:3:25::1 activate
 exit-address-family
~~~
~~~
R24#sh run | sec bgp
router bgp 520
 bgp router-id 10.100.255.24
 bgp log-neighbor-changes
 neighbor 2001:DB8:2:34::1 remote-as 301
 neighbor 2001:DB8:3:25::1 remote-as 520
 neighbor 2001:DB8:3:25::1 update-source Loopback0
 neighbor 2001:DB8:3:34::2 remote-as 2042
 !
 address-family ipv6
  neighbor 2001:DB8:2:34::1 activate
  neighbor 2001:DB8:3:25::1 activate
  neighbor 2001:DB8:3:34::2 activate
 exit-address-family
~~~
~~~
R25#sh run | sec bgp
router bgp 520
 bgp router-id 10.100.255.25
 bgp log-neighbor-changes
 neighbor AS520_ipv6 peer-group
 neighbor AS520_ipv6 remote-as 520
 neighbor AS520_ipv6 update-source Loopback0
 neighbor 2001:DB8:3:23::1 peer-group AS520_ipv6
 neighbor 2001:DB8:3:24::1 peer-group AS520_ipv6
 neighbor 2001:DB8:3:26::1 peer-group AS520_ipv6
 !
 address-family ipv6
  neighbor AS520_ipv6 route-reflector-client
  neighbor AS520_ipv6 next-hop-self
  neighbor 2001:DB8:3:23::1 activate
  neighbor 2001:DB8:3:24::1 activate
  neighbor 2001:DB8:3:26::1 activate
 exit-address-family
~~~
~~~
R26#sh run | sec bgp
router bgp 520
 bgp router-id 10.100.255.26
 bgp log-neighbor-changes
 neighbor 2001:DB8:3:25::1 remote-as 520
 neighbor 2001:DB8:3:25::1 update-source Loopback0
 neighbor 2001:DB8:3:35::2 remote-as 2042
 !
 address-family ipv6
  neighbor 2001:DB8:3:25::1 activate
  neighbor 2001:DB8:3:35::2 activate
 exit-address-family
~~~
~~~
R18#sh run | sec bgp
router bgp 2042
 bgp router-id 172.20.255.18
 bgp log-neighbor-changes
 bgp bestpath as-path multipath-relax
 neighbor 2001:DB8:3:34::1 remote-as 520
 neighbor 2001:DB8:3:35::1 remote-as 520
 !
 address-family ipv6
  network 2001:DB8:4:1::1/128
  neighbor 2001:DB8:3:34::1 activate
  neighbor 2001:DB8:3:35::1 activate
 exit-address-family
~~~
Проверим, eBGP и iBGP сесси поднялись.
~~~
R18#sh bgp ipv6 unicast summary
BGP router identifier 172.20.255.18, local AS number 2042
BGP table version is 4, main routing table version 4
3 network entries using 492 bytes of memory
3 path entries using 312 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1188 total bytes of memory
BGP activity 40/0 prefixes, 76/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:3:34::1
                4          520    1183    1177        4    0    0 17:47:30        2
2001:DB8:3:35::1
                4          520    1178    1182        4    0    0 17:47:35        0
~~~
~~~
R25#sh bgp ipv6 unicast summary
BGP router identifier 10.100.255.25, local AS number 520
BGP table version is 1, main routing table version 1
3 network entries using 492 bytes of memory
6 path entries using 624 bytes of memory
3/0 BGP path/bestpath attribute entries using 432 bytes of memory
5 BGP AS-PATH entries using 120 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1668 total bytes of memory
BGP activity 40/0 prefixes, 51/6 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
2001:DB8:3:23::1
                4          520    1224    1217        1    0    0 18:25:07        2
2001:DB8:3:24::1
                4          520    1226    1217        1    0    0 18:25:12        3
2001:DB8:3:26::1
                4          520    1221    1218        1    0    0 18:25:07        1
~~~

На R18 Санкт-Петербург и R14 R15 Москва, анонсированы Loopbackб в таблице маршрутизации они есть, првоерим связность Москвы и Санкт - Петербурга.
~~~
R14#sh run in lo0
interface Loopback0
 ip address 10.0.255.14 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
 ipv6 address 2001:DB8:10::14/128
 ipv6 enable
 ipv6 ospf 1 area 0
end
 !
 address-family ipv6
  network 2001:DB8:10::14/128
~~~
~~~
R15#sh run int lo0
interface Loopback0
 ip address 10.0.255.15 255.255.255.255
 ip ospf network point-to-point
 ip ospf 1 area 0
 ipv6 address 2001:DB8:10::15/128
 ipv6 enable
 ipv6 ospf 1 area 0
end
!
 address-family ipv6
  network 2001:DB8:10::15/128
~~~
~~~
R18#sh run int lo0
interface Loopback0
 ip address 172.20.255.18 255.255.255.255
 ipv6 address 2001:DB8:4:1::1/128
 ipv6 enable
end
 !
 address-family ipv6
  network 2001:DB8:4:1::1/128
~~~
~~~
R18#sh ipv6 route bgp
IPv6 Routing Table - default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:10::14/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
B   2001:DB8:10::15/128 [20/0]
     via FE80::A8BB:CCFF:FE01:8030, Ethernet0/2
~~~
~~~
R15#sh ipv6 route bgp
IPv6 Routing Table - default - 13 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       H - NHRP, I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea
       IS - ISIS summary, D - EIGRP, EX - EIGRP external, NM - NEMO
       ND - ND Default, NDp - ND Prefix, DCE - Destination, NDr - Redirect
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2, la - LISP alt
       lr - LISP site-registrations, ld - LISP dyn-eid, a - Application
B   2001:DB8:4:1::1/128 [20/0]
     via FE80::A8BB:CCFF:FE01:5000, Ethernet0/2

R15#ping 2001:DB8:4:1::1 source 2001:DB8:10::15
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:DB8:4:1::1, timeout is 2 seconds:
Packet sent with a source address of 2001:DB8:10::15
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
~~~
