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
Настройка маршрутизаторов в ISP Триада
```
R23#
router bgp 520
 bgp router-id 10.100.255.23
 bgp log-neighbor-changes
 neighbor 28.1.0.57 remote-as 101
```
```
R24#
router bgp 520
 bgp router-id 10.100.255.24
 bgp log-neighbor-changes
 neighbor 33.186.35.121 remote-as 301
 neighbor 56.23.124.54 remote-as 2042
```
```
R26#
router bgp 520
 bgp router-id 10.100.255.26
 bgp log-neighbor-changes
 neighbor 48.81.46.10 remote-as 2042
```
Настройка маршрутизатора ISP Cанкт-Петербурге
```
R18#
router bgp 2042
 bgp router-id 172.20.255.18
 bgp log-neighbor-changes
 neighbor 48.81.46.9 remote-as 520
 neighbor 56.23.124.53 remote-as 520
```
### Сессии поднялись
Москва
```
R14#sh ip bgp sum
BGP router identifier 10.0.255.14, local AS number 1001
BGP table version is 3, main routing table version 3
2 network entries using 280 bytes of memory
2 path entries using 160 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
1 BGP AS-PATH entries using 40 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 768 total bytes of memory
BGP activity 2/0 prefixes, 2/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
85.123.45.17    4          101     204     206        3    0    0 03:03:00        1
```
```
R15#sh bgp sum
BGP router identifier 10.0.255.15, local AS number 1001
BGP table version is 3, main routing table version 3
2 network entries using 280 bytes of memory
2 path entries using 160 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 752 total bytes of memory
BGP activity 2/0 prefixes, 2/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
185.15.145.53   4          301     202     203        3    0    0 03:00:43        1
```
Киторн
```
R22#sh bgp sum
BGP router identifier 193.12.255.22, local AS number 101
BGP table version is 4, main routing table version 4
3 network entries using 420 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1164 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
24.153.14.58    4          301     211     212        4    0    0 03:08:28        2
28.1.0.58       4          520     190     192        4    0    0 02:50:33        0
85.123.45.18    4         1001     208     206        4    0    0 03:04:44        1
```
Ламас
```
R21#sh bgp sum
BGP router identifier 37.24.255.21, local AS number 301
BGP table version is 4, main routing table version 4
3 network entries using 420 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1164 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
24.153.14.57    4          101     213     212        4    0    0 03:09:13        1
33.186.35.122   4          520     167     166        4    0    0 02:28:35        1
185.15.145.54   4         1001     204     204        4    0    0 03:02:15        1
```
Триада
```
R23#sh bgp sum
BGP router identifier 10.100.255.23, local AS number 520
BGP table version is 3, main routing table version 3
2 network entries using 280 bytes of memory
2 path entries using 160 bytes of memory
2/2 BGP path/bestpath attribute entries using 288 bytes of memory
2 BGP AS-PATH entries using 48 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 776 total bytes of memory
BGP activity 2/0 prefixes, 2/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
28.1.0.57       4          101     197     195        3    0    0 02:54:57        2
```
```
R24#sh bgp sum
BGP router identifier 10.100.255.24, local AS number 520
BGP table version is 4, main routing table version 4
3 network entries using 420 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1164 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
33.186.35.121   4          301     171     172        4    0    0 02:32:55        2
56.23.124.54    4         2042     153     153        4    0    0 02:14:47        1
```
```
R26#sh bgp sum
BGP router identifier 10.100.255.26, local AS number 520
BGP table version is 2, main routing table version 2
1 network entries using 140 bytes of memory
1 path entries using 80 bytes of memory
1/1 BGP path/bestpath attribute entries using 144 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 388 total bytes of memory
BGP activity 1/0 prefixes, 1/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
48.81.46.10     4         2042     150     148        2    0    0 02:13:00        1
```
Санкт-петербург
```
R18#sh bgp sum
BGP router identifier 172.20.255.18, local AS number 2042
BGP table version is 4, main routing table version 4
3 network entries using 420 bytes of memory
3 path entries using 240 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
2 BGP AS-PATH entries using 64 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1156 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
48.81.46.9      4          520     149     151        4    0    0 02:13:44        0
56.23.124.53    4          520     155     154        4    0    0 02:16:09        2
```
### Анонсируем Loopback интрефесы на маршрутизаторах в Москве и Санкт-Петербурге.
Москва
```
R14#show run | section bgp
router bgp 1001
 bgp router-id 10.0.255.14
 bgp log-neighbor-changes
 network 10.0.255.14 mask 255.255.255.255
 neighbor 85.123.45.17 remote-as 101
```
```
R15#sh run | section bgp
router bgp 1001
 bgp router-id 10.0.255.15
 bgp log-neighbor-changes
 network 10.0.255.15 mask 255.255.255.255
 neighbor 185.15.145.53 remote-as 301
```
Санкт-Петербург
```
R18#sh run | section bgp
router bgp 2042
 bgp router-id 172.20.255.18
 bgp log-neighbor-changes
 network 172.20.255.18 mask 255.255.255.255
 neighbor 48.81.46.9 remote-as 520
 neighbor 56.23.124.53 remote-as 520
```
