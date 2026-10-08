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
Проверим IPsec. Работает.
~~~
R18#sh crypto ikev2 sa
 IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
1         48.81.46.10/500       85.123.45.18/500      none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA256, Hash: SHA256, DH Grp:14, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3001 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status
3         56.23.124.53/500      185.15.145.54/500     none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA256, Hash: SHA256, DH Grp:14, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/61560 sec

 IPv6 Crypto IKEv2  SA

R18#sh crypto ipsec sa

interface: Tunnel1
    Crypto map tag: Tunnel1-head-0, local addr 48.81.46.10

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (48.81.46.10/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (85.123.45.18/255.255.255.255/47/0)
   current_peer 85.123.45.18 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 0, #pkts encrypt: 0, #pkts digest: 0
    #pkts decaps: 0, #pkts decrypt: 0, #pkts verify: 0
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 48.81.46.10, remote crypto endpt.: 85.123.45.18
     plaintext mtu 1466, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/3
     current outbound spi: 0xB4EFC028(3035611176)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0xC651E5C7(3327256007)
        transform: esp-3des esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 85, flow_id: SW:85, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4333406/608)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0xB4EFC028(3035611176)
        transform: esp-3des esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 86, flow_id: SW:86, sibling_flags 80000000, crypto map: Tunnel1-head-0
        sa timing: remaining key lifetime (k/sec): (4333406/608)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:

interface: Tunnel0
    Crypto map tag: Tunnel0-head-0, local addr 56.23.124.53

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (56.23.124.53/255.255.255.255/47/0)
   remote ident (addr/mask/prot/port): (185.15.145.54/255.255.255.255/47/0)
   current_peer 185.15.145.54 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 10, #pkts encrypt: 10, #pkts digest: 10
    #pkts decaps: 22, #pkts decrypt: 22, #pkts verify: 22
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 56.23.124.53, remote crypto endpt.: 185.15.145.54
     plaintext mtu 1466, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/3
     current outbound spi: 0x95241334(2502169396)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x745BD8A8(1952176296)
        transform: esp-3des esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 87, flow_id: SW:87, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4306377/793)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x95241334(2502169396)
        transform: esp-3des esp-sha256-hmac ,
        in use settings ={Transport, }
        conn id: 88, flow_id: SW:88, sibling_flags 80000000, crypto map: Tunnel0-head-0
        sa timing: remaining key lifetime (k/sec): (4306377/793)
        IV size: 8 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)
~~~
### Настроим Dmvpn и IPsec на маршрутизаторах R27(Лабытнанги) и R28(Чокурдах).
~~~
R27#
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
interface Tunnel0
 description DMVPN to Hub1
 ip address 10.0.252.3 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.0.252.1 185.15.145.54
 ip nhrp map multicast 185.15.145.54
 ip nhrp map 10.0.252.2 85.123.45.18
 ip nhrp map multicast 85.123.45.18
 ip nhrp network-id 100
 ip nhrp nhs 10.0.252.1 priority 1 cluster 1
 ip nhrp nhs 10.0.252.2 priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile IPSEC
end
~~~
~~~
R28#
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
interface Tunnel0
 description DMVPN to Hub1
 ip address 10.0.252.4 255.255.255.0
 no ip redirects
 ip mtu 1400
 ip nhrp map 10.0.252.1 185.15.145.54
 ip nhrp map multicast 185.15.145.54
 ip nhrp map 10.0.252.2 85.123.45.18
 ip nhrp map multicast 85.123.45.18
 ip nhrp network-id 100
 ip nhrp nhs 10.0.252.1 priority 1 cluster 1
 ip nhrp nhs 10.0.252.2 priority 2 cluster 1
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source Ethernet0/0
 tunnel mode gre multipoint
 tunnel protection ipsec profile IPSEC
end
~~~
