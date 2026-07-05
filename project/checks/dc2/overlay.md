# Проверка overlay EVPN

## Что проверяем

Проверяем overlay DC2: iBGP EVPN-соседства строятся по loopback-адресам между leaf/border leaf и spine route-reflector. Дополнительно проверяем MAC/IP routes, IMET routes и наличие IP-prefix routes для VRF `TENANT-DC2`.

## Выводы команд

Проверяем на spine, leaf и `bl-21`, что EVPN-соседства установлены, а в EVPN RIB присутствуют маршруты клиентов, IMET routes и IP-prefix routes для VLAN 210.

```
sp-21#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.21.0, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.21.1 4 65200            724       731    0    0 00:30:37 Estab   4      4
  10.0.21.2 4 65200            687       700    0    0 00:28:59 Estab   2      2
  10.0.21.3 4 65200            722       727    0    0 00:30:31 Estab   4      4
  10.0.21.4 4 65200            176       192    0    0 00:03:56 Estab   4      4
sp-21#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.21.0, local AS number 65200
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:10210 mac-ip 0050.7966.681d
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.4:10210 mac-ip 0050.7966.681d 192.168.210.13
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i
 * >      RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:10210 imet 10.0.21.4
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i
 * >      RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.4             -       100     0       i
sp-21#
```

```
sp-22#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.22.0, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.21.1 4 65200            722       729    0    0 00:30:34 Estab   4      4
  10.0.21.2 4 65200            688       697    0    0 00:29:05 Estab   2      2
  10.0.21.3 4 65200            723       731    0    0 00:30:39 Estab   4      4
  10.0.21.4 4 65200            179       198    0    0 00:04:00 Estab   4      4
sp-22#
sp-22#
sp-22#
sp-22#
sp-22#
sp-22#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.22.0, local AS number 65200
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:10210 mac-ip 0050.7966.681d
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.4:10210 mac-ip 0050.7966.681d 192.168.210.13
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i
 * >      RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:10210 imet 10.0.21.4
                                 10.0.21.4             -       100     0       i
 * >      RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i
 * >      RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i
 * >      RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i
 * >      RD: 10.0.21.4:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.4             -       100     0       i
sp-22#
```


```
le-21#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.21.1, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.21.0 4 65200           3040      3039    0    0 00:31:44 Estab   10     10
  10.0.22.0 4 65200           2935      2930    0    0 00:31:38 Estab   10     10
le-21#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.21.1, local AS number 65200
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 -                     -       -       0       i
 * >      RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.4:10210 mac-ip 0050.7966.681d
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.4:10210 mac-ip 0050.7966.681d
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.4:10210 mac-ip 0050.7966.681d 192.168.210.13
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.4:10210 mac-ip 0050.7966.681d 192.168.210.13
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.22.0
 * >      RD: 10.0.21.1:10210 imet 10.0.21.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.4:10210 imet 10.0.21.4
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.4:10210 imet 10.0.21.4
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.22.0
 * >      RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.4:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.4:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.4             -       100     0       i Or-ID: 10.0.21.4 C-LST: 10.0.22.0
le-21#
```


```
le-22#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.21.2, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.21.0 4 65200           1982      1969    0    0 01:12:24 Estab   8      8
  10.0.22.0 4 65200           1905      1903    0    0 01:20:50 Estab   8      8
le-22#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.21.2, local AS number 65200
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 * >      RD: 10.0.21.2:10210 imet 10.0.21.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.3:10210 imet 10.0.21.3
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 * >      RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.3             -       100     0       i Or-ID: 10.0.21.3 C-LST: 10.0.21.0
le-22#
```

```
le-23#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.21.3, local AS number 65200
Neighbor Status Codes: m - Under maintenance
  Neighbor  V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.21.0 4 65200           1952      1951    0    0 01:12:28 Estab   6      6
  10.0.22.0 4 65200           1911      1896    0    0 01:20:57 Estab   6      6
le-23#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.21.3, local AS number 65200
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 * >Ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 *  ec    RD: 10.0.21.1:10210 mac-ip 0050.7966.6818 192.168.210.11
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819
                                 -                     -       -       0       i
 * >      RD: 10.0.21.3:10210 mac-ip 0050.7966.6819 192.168.210.12
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.1:10210 imet 10.0.21.1
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.2:10210 imet 10.0.21.2
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.21.0
 * >      RD: 10.0.21.3:10210 imet 10.0.21.3
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.1:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.1             -       100     0       i Or-ID: 10.0.21.1 C-LST: 10.0.21.0
 * >Ec    RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.22.0
 *  ec    RD: 10.0.21.2:50200 ip-prefix 192.168.210.0/24
                                 10.0.21.2             -       100     0       i Or-ID: 10.0.21.2 C-LST: 10.0.21.0
 * >      RD: 10.0.21.3:50200 ip-prefix 192.168.210.0/24
                                 -                     -       -       0       i
le-23#
```
