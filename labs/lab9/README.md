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
 neighbor 2001:DB8:3:1::2 remote-as 301
 !
 address-family ipv6
  neighbor 2001:DB8:1:14::1 activate
  neighbor 2001:DB8:2:12::2 activate
  neighbor 2001:DB8:2:23::2 activate
  neighbor 2001:DB8:3:1::2 activate
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

