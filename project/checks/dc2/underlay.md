# Проверка underlay

## Что проверяем

Проверяем OSPF-соседства IPv4 underlay между leaf и spine DC2. Underlay строится по p2p-адресам `/31` и используется для достижимости loopback `/32`, необходимых для VTEP и overlay EVPN.

## Выводы команд

```
sp-21#show ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.21.2       1        default  0   FULL                   00:00:37    10.22.1.3       Ethernet2
10.0.21.1       1        default  0   FULL                   00:00:34    10.22.1.1       Ethernet1
10.0.21.3       1        default  0   FULL                   00:00:33    10.22.1.5       Ethernet3
sp-21#
```

```
sp-22#show ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.21.2       1        default  0   FULL                   00:00:29    10.22.2.3       Ethernet2
10.0.21.1       1        default  0   FULL                   00:00:33    10.22.2.1       Ethernet1
10.0.21.3       1        default  0   FULL                   00:00:34    10.22.2.5       Ethernet3
sp-22#
```

```
le-21#show ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.22.0       1        default  0   FULL                   00:00:31    10.22.2.0       Ethernet2
10.0.21.0       1        default  0   FULL                   00:00:32    10.22.1.0       Ethernet1
le-21#
```

```
le-22#show ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.22.0       1        default  0   FULL                   00:00:35    10.22.2.2       Ethernet2
10.0.21.0       1        default  0   FULL                   00:00:32    10.22.1.2       Ethernet1
le-22#
```

```
le-23#show ip ospf ne
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.22.0       1        default  0   FULL                   00:00:38    10.22.2.4       Ethernet2
10.0.21.0       1        default  0   FULL                   00:00:31    10.22.1.4       Ethernet1
le-23#
```
