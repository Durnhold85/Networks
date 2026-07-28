# Настроить iBGP в офисе Москва и в сети провайдера Триада для обеспечения полной IP-связности всех сетей.

# План работ:

1. Настроить iBGP в офисом Москва между маршрутизаторами R14 и R15.
2. Настроить iBGP в провайдере Триада, с использованием RR.
3. Настроить офис Москва так, чтобы приоритетным провайдером стал Ламас.
4. Настроить офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
5. Все сети в лабораторной работе должны иметь IP связность.

## Схема стенда.

![](BGP.png)

### Настроим IBGP в офисе Москва, а так же сделаем приоритетным провайдером ISP Ламас.

![](moscow_ibgp.png)
Настроим IBGP сессию между R14 и R15, на R15 повысим значение атрибута local preference сделав его маршруты приоритетнее для исходящего трафика.
Создадим route-map LOCAL_PREF_LAMAS и привяжем её к соседу eBGP neighbor 185.15.145.53

```
R15#
route-map LOCAL_PREF_LAMAS permit 10
 set local-preference 200
```
```
router bgp 1001
 bgp router-id 10.0.255.15
 bgp log-neighbor-changes
 network 10.0.255.15 mask 255.255.255.255
 neighbor 10.0.255.14 remote-as 1001
 neighbor 10.0.255.14 update-source Loopback0
 neighbor 10.0.255.14 next-hop-self
 neighbor 185.15.145.53 remote-as 301
 neighbor 185.15.145.53 route-map LOCAL_PREF_LAMAS in
```
На R14 настроим IBGP сессию с R15 и изменим значение атрибута as-path, добавим наш номер AS в as-path чтобы для провайдера Киторн полученные от нам маршруты были менее приоритетными.
```
R14#
route-map SET_AS_PATH_PREPEND permit 10
 set as-path prepend 1001 1001 1001
```
```
router bgp 1001
 bgp router-id 10.0.255.14
 bgp log-neighbor-changes
 network 10.0.255.14 mask 255.255.255.255
 neighbor 10.0.255.15 remote-as 1001
 neighbor 10.0.255.15 update-source Loopback0
 neighbor 10.0.255.15 next-hop-self
 neighbor 85.123.45.17 remote-as 101
 neighbor 85.123.45.17 route-map SET_AS_PATH_PREPEND out
```
