# Настроить фильтрацию в офисах Москва и С.-Петербург.

# План работ:

1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика(As-path).
2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика(Prefix-list).
3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.
5. Все сети в лабораторной работе должны иметь IP связность.

## Схема стенда.

![](BGP.png)

### Настроим R14 и R15 в Москве так чтобы не было транзитного трафика. Используем filter-list.
```
R14#
ip as-path access-list 55 permit ^$
!
router bgp 1001
 bgp router-id 10.0.255.14
 neighbor 85.123.45.17 filter-list 55 out
```


```
R15#
ip as-path access-list 55 permit ^$
!
router bgp 1001
 bgp router-id 10.0.255.15
 neighbor 185.15.145.53 filter-list 55 out
```
Видимо что провайдеры получают только наши локальные префиксы.
```
R22#sh ip bgp neighbors 85.123.45.18 received-routes
BGP table version is 54, local router ID is 193.12.255.22
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   10.0.254.0/30    85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.4/30    85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.8/30    85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.12/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.16/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.20/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.24/30   85.123.45.18                           0 1001 1001 1001 1001 ?
     Network          Next Hop            Metric LocPrf Weight Path
 *   10.0.254.28/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.32/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.254.36/30   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.4/32    85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.5/32    85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.12/32   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.13/32   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.14/32   85.123.45.18             0             0 1001 1001 1001 1001 i
 *   10.0.255.15/32   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.19/32   85.123.45.18                           0 1001 1001 1001 1001 ?
 *   10.0.255.20/32   85.123.45.18                           0 1001 1001 1001 1001 ?
     Network          Next Hop            Metric LocPrf Weight Path

Total number of prefixes 18

```
```
R21#sh ip bgp neighbors 185.15.145.54 received-routes
BGP table version is 96, local router ID is 37.24.255.21
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.254.0/30    185.15.145.54            0             0 1001 ?
 *>  10.0.254.4/30    185.15.145.54            0             0 1001 ?
 *>  10.0.254.8/30    185.15.145.54           20             0 1001 ?
 *>  10.0.254.12/30   185.15.145.54            0             0 1001 ?
 *>  10.0.254.16/30   185.15.145.54           30             0 1001 ?
 *>  10.0.254.20/30   185.15.145.54           20             0 1001 ?
 *>  10.0.254.24/30   185.15.145.54           20             0 1001 ?
 *>  10.0.254.28/30   185.15.145.54           20             0 1001 ?
 *>  10.0.254.32/30   185.15.145.54           20             0 1001 ?
 *>  10.0.254.36/30   185.15.145.54           20             0 1001 ?
 *>  10.0.255.4/32    185.15.145.54           21             0 1001 ?
 *>  10.0.255.5/32    185.15.145.54           21             0 1001 ?
 *>  10.0.255.12/32   185.15.145.54           11             0 1001 ?
 *>  10.0.255.13/32   185.15.145.54           11             0 1001 ?
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.0.255.14/32   185.15.145.54           21             0 1001 ?
 *>  10.0.255.15/32   185.15.145.54            0             0 1001 ?
 *>  10.0.255.19/32   185.15.145.54           31             0 1001 ?
 *>  10.0.255.20/32   185.15.145.54           11             0 1001 ?

Total number of prefixes 18
```
### Настроим R18 в Санкт-Петербурге так чтобы не было транзитного трафика. Используем prefix-list и route-map.

```
R18#
ip prefix-list LOCAL seq 5 permit 172.20.0.0/16 le 32
!
route-map TO_TRIADA permit 10
 match ip address prefix-list LOCAL
!
router bgp 2042
 bgp router-id 172.20.255.18
 neighbor 48.81.46.9 route-map TO_TRIADA out
 neighbor 56.23.124.53 route-map TO_TRIADA out

```

### Настроим R22 ISP Киторн так чтобы в сторону R14 (Москва) приходил только default route.

```
R22#
ip prefix-list DEFAULT seq 5 permit 0.0.0.0/0
!
route-map ONLY_DEFAULT permit 10
 match ip address prefix-list DEFAULT
!
router bgp 101
 bgp router-id 193.12.255.22
 neighbor 85.123.45.18 route-map ONLY_DEFAULT out
```
Проверим на R14, по bgp от соседа приходит только default.
```
R14#sh ip bgp neighbors 85.123.45.17 received-routes
BGP table version is 100, local router ID is 10.0.255.14
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 r   0.0.0.0          85.123.45.17                           0 101 i

Total number of prefixes 1
```
### Настроим R21 ISP Ламас так чтобы в сторону R15 (Москва) приходил только default route и префиксы AS 2042 (Санкт-Петербург).
```
R21#
ip as-path access-list 10 permit _2042$
!
router bgp 301
 bgp router-id 37.24.255.21
 neighbor 185.15.145.54 default-originate
 neighbor 185.15.145.54 filter-list 10 out
```
Проверим на R15, по bgp от соседа приходит только default и префиксы с AS2042

```
R15#sh ip bgp neighbors 185.15.145.53 received-routes
BGP table version is 99, local router ID is 10.0.255.15
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *   0.0.0.0          185.15.145.53                          0 301 i
 *   172.20.30.0/24   185.15.145.53                          0 301 520 2042 ?
 *   172.20.40.0/24   185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.0/30  185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.4/30  185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.8/30  185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.12/30 185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.16/30 185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.20/30 185.15.145.53                          0 301 520 2042 ?
 *   172.20.254.24/30 185.15.145.53                          0 301 520 2042 ?
 *   172.20.255.9/32  185.15.145.53                          0 301 520 2042 ?
 *   172.20.255.10/32 185.15.145.53                          0 301 520 2042 ?
 *   172.20.255.16/32 185.15.145.53                          0 301 520 2042 ?
 *   172.20.255.16/30 185.15.145.53                          0 301 520 2042 ?
     Network          Next Hop            Metric LocPrf Weight Path
 *   172.20.255.18/32 185.15.145.53                          0 301 520 2042 ?
 *   172.20.255.32/32 185.15.145.53                          0 301 520 2042 ?

Total number of prefixes 16
```
