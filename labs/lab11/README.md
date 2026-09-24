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
