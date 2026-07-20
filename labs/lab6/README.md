# Настроить BGP между автономными системами и организовать доступность между офисами Москва и С.-Петербург.

# План работ:

1. Настроить eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас.
2. Настроить eBGP между провайдерами Киторн и Ламас.
3. Настроить eBGP между Ламас и Триада.
4. Настроить eBGP между офисом С.-Петербург и провайдером Триада.
5. Организовать IP доступность между пограничными роутерами офисов Москва и С.-Петербург.

## Схема стенда.

![](BGP.png)

### Настроим все eBGP сессии между маршрутизаторами.

Настройка маршрутизаторов в Москве
```
R14#
router bgp 1001
 bgp router-id 10.0.255.14
 bgp log-neighbor-changes
 neighbor 85.123.45.17 remote-as 101
```
```
R15#
router bgp 1001
 bgp router-id 10.0.255.15
 bgp log-neighbor-changes
 network 10.0.255.15 mask 255.255.255.255
 neighbor 185.15.145.53 remote-as 301
```
Настройка маршрутизатора ISP Киторн
```
R22#
router bgp 101
 bgp router-id 193.12.255.22
 bgp log-neighbor-changes
 neighbor 24.153.14.58 remote-as 301
 neighbor 28.1.0.58 remote-as 520
 neighbor 85.123.45.18 remote-as 1001
```
Настройка маршрутизатора ISP Ламас
```
R21#
router bgp 301
 bgp router-id 37.24.255.21
 bgp log-neighbor-changes
 neighbor 24.153.14.57 remote-as 101
 neighbor 33.186.35.122 remote-as 520
 neighbor 185.15.145.54 remote-as 1001
```
