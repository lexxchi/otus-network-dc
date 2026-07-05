# Проверка overlay EVPN

## Что проверяем

Проверяем разделение underlay и overlay: IPv4 BGP по p2p-соседствам распространяет только loopback `/32`, а EVPN-соседства строятся по loopback-адресам. Дополнительно проверяем MAC/IP routes, IMET routes и наличие маршрутов в VRF TENANT-1.

## Выводы команд

Проверяем на spine, что в IPv4 BGP RIB присутствуют loopback `/32`, а EVPN-соседства строятся с leaf/border leaf по loopback-адресам.

```text
sp-1#show bgp ipv4 unicast su
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1 4 65101           1420      1424    0    0 01:00:20 Estab   1      1
  10.2.1.3 4 65102           1424      1421    0    0 01:00:22 Estab   1      1
  10.2.1.5 4 65103           1436      1426    0    0 01:00:25 Estab   1      1
  10.2.1.7 4 65104           1395      1415    0    0 00:59:35 Estab   1      1
  10.2.1.9 4 65105           1430      1430    0    0 01:00:28 Estab   1      1
sp-1#show bgp ipv4 unicast
BGP routing table information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Route status codes: s - suppressed contributor, * - valid, > - active, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
                    % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI Origin Validation codes: V - valid, I - invalid, U - unknown
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  AIGP       LocPref Weight  Path
 * >      10.0.1.0/32            -                     -       -          -       0       i
 * >      10.0.1.1/32            10.2.1.1              0       -          100     0       65101 i
 * >      10.0.1.2/32            10.2.1.3              0       -          100     0       65102 i
 * >      10.0.1.3/32            10.2.1.5              0       -          100     0       65103 i
 * >      10.0.1.4/32            10.2.1.7              0       -          100     0       65104 i
 * >      10.0.1.5/32            10.2.1.9              0       -          100     0       65105 i
sp-1#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.1 4 65101           1579      1511    0    0 01:01:05 Estab   10     10
  10.0.1.2 4 65102           1571      1524    0    0 01:01:07 Estab   10     10
  10.0.1.3 4 65103           1554      1604    0    0 01:01:14 Estab   9      9
  10.0.1.4 4 65104           1531      1572    0    0 01:00:24 Estab   6      6
  10.0.1.5 4 65105           1585      1606    0    0 01:01:17 Estab   10     10
```


```text
le-1#show bgp evpn summary
BGP summary information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.0.1.0 4 65001           2425      2376    0    0 00:58:54 Estab   37     37
  10.0.2.0 4 65001           1457      1481    0    0 00:58:59 Estab   37     37


le-1#show bgp evpn route-type mac-ip
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.1:10010 mac-ip 0050.7966.6807
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10010 mac-ip 0050.7966.6807
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10010 mac-ip 0050.7966.6807
                                 10.0.1.2              -       100     0       65001 65102 i
 * >Ec    RD: 10.0.1.2:10010 mac-ip 0050.7966.6807 192.168.10.11
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10010 mac-ip 0050.7966.6807 192.168.10.11
                                 10.0.1.2              -       100     0       65001 65102 i
 * >      RD: 10.0.1.1:10020 mac-ip 0050.7966.680b
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680b
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680b
                                 10.0.1.2              -       100     0       65001 65102 i
 * >Ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680b 192.168.20.11
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680b 192.168.20.11
                                 10.0.1.2              -       100     0       65001 65102 i
 * >      RD: 10.0.1.1:10020 mac-ip 0050.7966.680c
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680c
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 mac-ip 0050.7966.680c
                                 10.0.1.2              -       100     0       65001 65102 i
 * >      RD: 10.0.1.1:10020 mac-ip 0050.7966.680c 192.168.20.12
                                 -                     -       -       0       i
 * >      RD: 10.0.1.1:10020 mac-ip 5001.000f.0001
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10020 mac-ip 5001.000f.0001
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 mac-ip 5001.000f.0001
                                 10.0.1.2              -       100     0       65001 65102 i
 * >      RD: 10.0.1.1:10020 mac-ip 5001.000f.0001 192.168.20.112
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10020 mac-ip 5001.000f.0001 192.168.20.112
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 mac-ip 5001.000f.0001 192.168.20.112
                                 10.0.1.2              -       100     0       65001 65102 i
 * >Ec    RD: 10.0.1.3:10020 mac-ip 5001.0011.0001
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10020 mac-ip 5001.0011.0001
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.5:10020 mac-ip 5001.0011.0001
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10020 mac-ip 5001.0011.0001
                                 10.0.1.5              -       100     0       65001 65105 i
 * >Ec    RD: 10.0.1.3:10020 mac-ip 5001.0011.0001 192.168.20.44
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10020 mac-ip 5001.0011.0001 192.168.20.44
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.5:10020 mac-ip 5001.0011.0001 192.168.20.44
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10020 mac-ip 5001.0011.0001 192.168.20.44
                                 10.0.1.5              -       100     0       65001 65105 i
le-1#


le-1#show bgp evpn route-type imet
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >      RD: 10.0.1.1:10010 imet 10.0.1.1
                                 -                     -       -       0       i
 * >      RD: 10.0.1.1:10020 imet 10.0.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.2:10010 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10010 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65001 65102 i
 * >Ec    RD: 10.0.1.2:10020 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65001 65102 i
 *  ec    RD: 10.0.1.2:10020 imet 10.0.1.2
                                 10.0.1.2              -       100     0       65001 65102 i
 * >Ec    RD: 10.0.1.3:10010 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10010 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.3:10020 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10020 imet 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.4:10010 imet 10.0.1.4
                                 10.0.1.4              -       100     0       65001 65104 i
 *  ec    RD: 10.0.1.4:10010 imet 10.0.1.4
                                 10.0.1.4              -       100     0       65001 65104 i
 * >Ec    RD: 10.0.1.4:10020 imet 10.0.1.4
                                 10.0.1.4              -       100     0       65001 65104 i
 *  ec    RD: 10.0.1.4:10020 imet 10.0.1.4
                                 10.0.1.4              -       100     0       65001 65104 i
 * >Ec    RD: 10.0.1.5:10010 imet 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10010 imet 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
 * >Ec    RD: 10.0.1.5:10020 imet 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10020 imet 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
le-1#

le-1#show ip route vrf TENANT-1

VRF: TENANT-1
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B - Other BGP Routes,
       B I - iBGP, B E - eBGP, R - RIP, I L1 - IS-IS level 1,
       I L2 - IS-IS level 2, O3 - OSPFv3, A B - BGP Aggregate,
       A O - OSPF Summary, NG - Nexthop Group Static Route,
       V - VXLAN Control Service, M - Martian,
       DH - DHCP client installed default route,
       DP - Dynamic Policy Route, L - VRF Leaked,
       G  - gRIBI, RC - Route Cache Route

Gateway of last resort:
 B E      0.0.0.0/0 [200/0] via VTEP 10.0.1.4 VNI 50000 router-mac 50:01:00:02:f6:c5 local-interface Vxlan1

 B E      192.168.10.11/32 [200/0] via VTEP 10.0.1.2 VNI 50000 router-mac 50:01:00:27:03:91 local-interface Vxlan1
 C        192.168.10.0/24 is directly connected, Vlan10
 B E      192.168.20.11/32 [200/0] via VTEP 10.0.1.2 VNI 50000 router-mac 50:01:00:27:03:91 local-interface Vxlan1
 B E      192.168.20.44/32 [200/0] via VTEP 10.0.1.5 VNI 50000 router-mac 50:01:00:80:9b:62 local-interface Vxlan1
                                   via VTEP 10.0.1.3 VNI 50000 router-mac 50:01:00:ca:7a:8d local-interface Vxlan1
 B E      192.168.20.112/32 [200/0] via VTEP 10.0.1.2 VNI 50000 router-mac 50:01:00:27:03:91 local-interface Vxlan1
 C        192.168.20.0/24 is directly connected, Vlan20
 B E      203.0.113.0/30 [200/0] via VTEP 10.0.1.4 VNI 50000 router-mac 50:01:00:02:f6:c5 local-interface Vxlan1

le-1#
```
