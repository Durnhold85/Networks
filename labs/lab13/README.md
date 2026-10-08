# Настроить GRE поверх IPSec между офисами Москва и Санкт-Петербург, а также DMVPN поверх IPSec между Москвой, Чокурдахом и Лабытнанги.

# План работ:

1. Настроить GRE поверх IPSec между офисами Москва и С.-Петербург.
2. Настроить DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги.

### Настроим IPsec между Москвой и Санкт-Петербургом.
На маршрутизаторах в Москве также подготовим настройки для DMVPN
~~~
R14#
!
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
crypto ikev2 policy IKEV2-Policy
 proposal IKEV2-PROP
crypto ikev2 keyring DMVPN-KEYS
 peer ANY-SPOKE
  address 0.0.0.0 0.0.0.0
  pre-shared-key cisco123
 !
crypto ikev2 profile IKEV2-PROF
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEYS
 dpd 10 2 on-demand
crypto ipsec transform-set GRE esp-3des esp-sha256-hmac
 mode transport
crypto ipsec profile IPSEC
 set transform-set GRE
 set ikev2-profile IKEV2-PROF
!
interface Tunnel1
 ip address 10.11.0.5 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 85.123.45.18
 tunnel destination 48.81.46.10
 tunnel protection ipsec profile IPSEC
end
!
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
 tunnel protection ipsec profile IPSEC
end
~~~
~~~
R15#
!crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
crypto ikev2 policy IKEV2-Policy
 proposal IKEV2-PROP
crypto ikev2 keyring DMVPN-KEYS
 peer ANY-SPOKE
  address 0.0.0.0 0.0.0.0
  pre-shared-key cisco123
 !
crypto ikev2 profile IKEV2-PROF
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEYS
 dpd 10 2 on-demand
crypto ipsec transform-set GRE esp-3des esp-sha256-hmac
 mode transport
crypto ipsec profile IPSEC
 set transform-set GRE
 set ikev2-profile IKEV2-PROF
!
interface Tunnel0
 ip address 10.11.0.1 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 185.15.145.54
 tunnel destination 56.23.124.53
 tunnel protection ipsec profile IPSEC
end
!
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
 tunnel protection ipsec profile IPSEC
end
~~~
~~~
R18#
crypto ikev2 proposal IKEV2-PROP
 encryption aes-cbc-256
 integrity sha256
 group 14
crypto ikev2 policy IKEV2-Policy
 match address local 48.81.46.10
 match address local 56.23.124.53
 proposal IKEV2-PROP
crypto ikev2 keyring IKEV2-KEY
 peer R14
  address 85.123.45.18
  pre-shared-key local 6 cisco123
  pre-shared-key remote 6 cisco123
 !
 peer R15
  address 185.15.145.54
  pre-shared-key local 6 cisco123
  pre-shared-key remote 6 cisco123
 !
crypto ikev2 profile IKEV2-PROF
 match identity remote address 85.123.45.18 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local IKEV2-KEY
crypto ikev2 profile IKEV2-PROF_2
 match identity remote address 185.15.145.54 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local IKEV2-KEY
 dpd 10 2 on-demand
crypto ipsec transform-set GRE esp-3des esp-sha256-hmac
 mode transport
crypto ipsec profile IPSEC
 set transform-set GRE
 set ikev2-profile IKEV2-PROF
crypto ipsec profile IPSEC_2
 set transform-set GRE
 set ikev2-profile IKEV2-PROF_2
!
interface Tunnel0
 ip address 10.11.0.2 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 56.23.124.53
 tunnel destination 185.15.145.54
 tunnel protection ipsec profile IPSEC_2
!
interface Tunnel1
 ip address 10.11.0.6 255.255.255.252
 ip mtu 1400
 ip tcp adjust-mss 1360
 tunnel source 48.81.46.10
 tunnel destination 85.123.45.18
 tunnel protection ipsec profile IPSEC
~~~
