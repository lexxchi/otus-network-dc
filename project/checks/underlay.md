# Проверка underlay

## Что проверяем

Проверяем eBGP-соседства IPv4 underlay между leaf/border leaf и spine. Underlay-сессии строятся по p2p-адресам `/31` и используются для распространения loopback `/32`, необходимых для VTEP и overlay EVPN.

## Выводы команд

```text
le-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.0 4 65001         103156    105116    0    0 01:01:32 Estab   5      5
  10.2.2.0 4 65001         101948    103640    0    0 01:01:32 Estab   5      5
le-1#


le-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 65001           2197      2191    0    0 01:01:50 Estab   5      5
  10.2.2.2 4 65001           1718      1707    0    0 01:01:50 Estab   5      5
le-2#


le-3#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.4 4 65001           9961      9694    0    0 01:02:01 Estab   5      5
  10.2.2.4 4 65001           9402      9272    0    0 01:02:01 Estab   5      5
le-3#

le-4#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.5, local AS number 65105
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.8 4 65001           2119      2114    0    0 01:02:19 Estab   5      5
  10.2.2.8 4 65001           1703      1705    0    0 01:02:18 Estab   5      5
le-4#

bl-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.4, local AS number 65104
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.6 4 65001           1883      1861    0    0 01:01:47 Estab   5      5
  10.2.2.6 4 65001           1673      1676    0    0 01:01:47 Estab   5      5
bl-1#

sp-1#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1 4 65101           1478      1480    0    0 01:02:46 Estab   1      1
  10.2.1.3 4 65102           1480      1477    0    0 01:02:48 Estab   1      1
  10.2.1.5 4 65103           1492      1484    0    0 01:02:51 Estab   1      1
  10.2.1.7 4 65104           1452      1471    0    0 01:02:01 Estab   1      1
  10.2.1.9 4 65105           1487      1487    0    0 01:02:54 Estab   1      1
sp-1#

sp-2#show ip bgp summary
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.2.1 4 65101           1481      1477    0    0 01:03:01 Estab   1      1
  10.2.2.3 4 65102           1477      1487    0    0 01:03:03 Estab   1      1
  10.2.2.5 4 65103           1481      1482    0    0 01:03:06 Estab   1      1
  10.2.2.7 4 65104           1464      1468    0    0 01:02:16 Estab   1      1
  10.2.2.9 4 65105           1488      1497    0    0 01:03:08 Estab   1      1
sp-2#
```
