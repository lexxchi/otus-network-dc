### Underlay. BGP


### Цель
настроить BGP для Underlay сети.

### Шаги
1. Настроите BGP в Underlay сети, для IP связанности между всеми сетевыми устройствами. iBGP или eBGP - решать вам!
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедитесь в наличии IP связанности между устройствами в BGP домене

### Ход выполнения
#### 1. Используем схему из [лабораторной работы #3](../lab-03/)

Зачистим конфигурацию от IS-IS на всех аристах:

```
interface Ethernet1-3
   no isis enable 49
   no isis bfd
   no isis network point-to-point
   no isis authentication mode md5
   no isis authentication key
   default isis bfd

interface Loopback0
   no isis enable 49

no router isis 49
```

#### 2. Зададим минимальную конфигурацию eBGP

Все spine поместим в одну AS 65001, при этом каждый leaf будет в своей отдельной AS 65101-65103.
В итоге получим такую вот схему сети

![eve-ng-scheme.png](eve-ng-scheme.png)



Со стороны spine пропишем автоподнятие сессии через peer group. Минимальная конфигурация на примере sp-1

```
sp-1#sh run | s bgp
router bgp 65001
   router-id 10.0.1.0
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.2.0.0/16 peer-group LEAFS peer-filter PF_LEAFS
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   !
   address-family ipv4
      neighbor LEAFS activate
      network 10.0.1.0/32
      redistribute connected
sp-1#

sp-1#sh run | b peer-filter PF_LEAFS
peer-filter PF_LEAFS
   10 match as-range 65001-65109 result accept
```

минимальный набор команд для поднятия связности на примере le-3


```
le-3#sh run | s bgp
router bgp 65103
   router-id 10.0.1.3
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES bfd
   neighbor 10.2.1.4 peer group SPINES
   neighbor 10.2.2.4 peer group SPINES
   !
   address-family ipv4
      neighbor SPINES activate
      network 10.0.1.3/32
      redistribute connected
le-3#
```

#### 3. Проверим IP связность

Проверим, что всё работает как ожидается. Поправим какие-то моменты и применим конфигурацию на остальных устройствах.

Видим, что поднялась сессия с le-1, но не с le-2. Это нормально, на le-2 конфигурация ещё не применена.
```
le-3#sh ip bgp su
BGP summary information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.4         4  65001            244       246    0    0 00:05:07 Estab   1      1
  10.2.2.4         4  65001            212       214    0    0 00:01:09 Active
```

BFD работает:
```
le-3#show bfd peer
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.1.4  2914318259  3503967829        Ethernet1(14)  normal   06/08/26 19:47
10.2.2.4  3080550595           0        Ethernet2(15)  normal   06/08/26 19:40

         LastDown                LastDiag    State
-------------------- ----------------------- -----
   06/08/26 19:47           No Diagnostic       Up
   06/08/26 19:51       Nbr Signaled Down     Down

le-3#
```

Лупбек sp-1 прилетает и доступен, всё хорошо
```
le-3#sh ip bgp
BGP routing table information for VRF default
Router identifier 10.0.1.3, local AS number 65103
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.0.1.0/32            10.2.1.4              0       100     0       65001 i
 * >     10.0.1.3/32            -                     0       0       -       i
le-3#ping 10.0.1.0
PING 10.0.1.0 (10.0.1.0) 72(100) bytes of data.
80 bytes from 10.0.1.0: icmp_seq=1 ttl=64 time=5.84 ms
80 bytes from 10.0.1.0: icmp_seq=2 ttl=64 time=8.56 ms
80 bytes from 10.0.1.0: icmp_seq=3 ttl=64 time=6.34 ms
80 bytes from 10.0.1.0: icmp_seq=4 ttl=64 time=5.37 ms
80 bytes from 10.0.1.0: icmp_seq=5 ttl=64 time=5.89 ms

--- 10.0.1.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 5.373/6.404/8.569/1.125 ms, ipg/ewma 7.591/6.073 ms
le-3#
```

Можно разлить конфигурацию на остальные устройства, заменив переменные.
Проверим полную картину после разливки конфигурации.

На sp-1 видим все сессии с лифами
```
sp-1#show ip bgp su
BGP summary information for VRF default
Router identifier 10.0.1.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.1         4  65101            769       770    0    0 00:38:09 Estab   1      1
  10.2.1.3         4  65102            759       760    0    0 00:37:40 Estab   1      1
  10.2.1.5         4  65103           1121      1120    0    0 00:55:40 Estab   1      1
```
На sp-2 так же видим все сессии с лифами
```
sp-2#show ip bgp su
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor         V  AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.2.1         4  65101            774       773    0    0 00:38:19 Estab   1      1
  10.2.2.3         4  65102            764       763    0    0 00:37:50 Estab   1      1
  10.2.2.5         4  65103            755       754    0    0 00:37:21 Estab   1      1
sp-2#
```
Проверим на le-2, так же видим все лупбэки и видим что есть ECMP

```
le-2#show ip bgp
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, # - not installed, E - ECMP head, e - ECMP
                    S - Stale, c - Contributing to ECMP, b - backup, L - labeled-unicast
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

         Network                Next Hop            Metric  LocPref Weight  Path
 * >     10.0.1.0/32            10.2.1.2              0       100     0       65001 i
 * >Ec   10.0.1.1/32            10.2.1.2              0       100     0       65001 65101 i
 *  ec   10.0.1.1/32            10.2.2.2              0       100     0       65001 65101 i
 * >     10.0.1.2/32            -                     0       0       -       i
 * >Ec   10.0.1.3/32            10.2.1.2              0       100     0       65001 65103 i
 *  ec   10.0.1.3/32            10.2.2.2              0       100     0       65001 65103 i
 * >     10.0.2.0/32            10.2.2.2              0       100     0       65001 i
le-2#
```

Проверим, что с le-3 есть пинг до всех лупбеков других лифов:

```
le-3#ping 10.0.1.1 source loopback 0
PING 10.0.1.1 (10.0.1.1) from 10.0.1.3 : 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=63 time=10.5 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=63 time=8.01 ms
80 bytes from 10.0.1.1: icmp_seq=3 ttl=63 time=7.41 ms
80 bytes from 10.0.1.1: icmp_seq=4 ttl=63 time=8.07 ms
80 bytes from 10.0.1.1: icmp_seq=5 ttl=63 time=8.92 ms

--- 10.0.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 43ms
rtt min/avg/max/mdev = 7.415/8.604/10.591/1.105 ms, ipg/ewma 10.817/9.588 ms
le-3#ping 10.0.1.2 source loopback 0
PING 10.0.1.2 (10.0.1.2) from 10.0.1.3 : 72(100) bytes of data.
80 bytes from 10.0.1.2: icmp_seq=1 ttl=63 time=37.0 ms
80 bytes from 10.0.1.2: icmp_seq=2 ttl=63 time=34.7 ms
80 bytes from 10.0.1.2: icmp_seq=3 ttl=63 time=28.6 ms
80 bytes from 10.0.1.2: icmp_seq=4 ttl=63 time=22.8 ms
80 bytes from 10.0.1.2: icmp_seq=5 ttl=63 time=8.08 ms

--- 10.0.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 67ms
rtt min/avg/max/mdev = 8.087/26.255/37.031/10.345 ms, pipe 4, ipg/ewma 16.895/30.860 ms
le-3#
```

ECMP так же работает, если вывести 1 любой спайн из работы, пинг продолжает идти


#### 4. Применим рекомендации

1. На спайнах можно уменьшить range для ASN, тк на лифах AS в диапазоне 65101-65103

```
sp-2(config)#no peer-filter PF_LEAFS
sp-2(config)#peer-filter PF_LEAFS
sp-2(config-peer-filter-PF_LEAFS)#match as-range 65101-65103 result accept
```
После сlear ip bgp сессия  снова поднялась, значит всё корректно


### Листинг конфигов устройств

sp-1
```
! Command: show running-config
! device: sp-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
alias c configure terminal
alias ii show interfaces descr
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname sp-1
!
spanning-tree mode mstp
!
interface Ethernet1
   description 1@le-1
   mtu 8000
   no switchport
   ip address 10.2.1.0/31
!
interface Ethernet2
   description 1@le-2
   mtu 8000
   no switchport
   ip address 10.2.1.2/31
   no ip ospf neighbor bfd
!
interface Ethernet3
   description 1@le-3
   mtu 8000
   no switchport
   ip address 10.2.1.4/31
   no ip ospf neighbor bfd
!
interface Ethernet4
   shutdown
!
interface Ethernet5
   shutdown
!
interface Ethernet6
   shutdown
!
interface Ethernet7
   shutdown
!
interface Ethernet8
   shutdown
!
interface Loopback0
   ip address 10.0.1.0/32
!
interface Management1
!
ip routing
!
peer-filter PF_LEAFS
   10 match as-range 65101-65103 result accept
!
router bgp 65001
   router-id 10.0.1.0
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.2.0.0/16 peer-group LEAFS peer-filter PF_LEAFS
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   !
   address-family ipv4
      neighbor LEAFS activate
      network 10.0.1.0/32
      redistribute connected
!
end
```
sp-2
```
! Command: show running-config
! device: sp-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
alias c configure terminal
alias ii show interfaces descr
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname sp-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description 2@le-1
   mtu 8000
   no switchport
   ip address 10.2.2.0/31
   no ip ospf neighbor bfd
!
interface Ethernet2
   description 2@le-2
   mtu 8000
   no switchport
   ip address 10.2.2.2/31
   no ip ospf neighbor bfd
!
interface Ethernet3
   description 2@le-3
   mtu 8000
   no switchport
   ip address 10.2.2.4/31
   no ip ospf neighbor bfd
!
interface Ethernet4
   shutdown
!
interface Ethernet5
   shutdown
!
interface Ethernet6
   shutdown
!
interface Ethernet7
   shutdown
!
interface Ethernet8
   shutdown
!
interface Loopback0
   ip address 10.0.2.0/32
!
interface Management1
!
ip routing
!
peer-filter PF_LEAFS
   10 match as-range 65101-65103 result accept
!
router bgp 65001
   router-id 10.0.2.0
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.2.0.0/16 peer-group LEAFS peer-filter PF_LEAFS
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   !
   address-family ipv4
      neighbor LEAFS activate
      network 10.0.2.0/32
      redistribute connected
!
end
```
le-1
```
! Command: show running-config
! device: le-1 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
alias c configure terminal
alias ii show interfaces descr
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname le-1
!
spanning-tree mode mstp
!
aaa authorization exec default local
!
interface Ethernet1
   description 1@sp-1
   mtu 8000
   no switchport
   ip address 10.2.1.1/31
!
interface Ethernet2
   description 1@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.1/31
!
interface Ethernet3
   description eth0@cl-1
   mtu 8000
!
interface Ethernet4
   shutdown
!
interface Ethernet5
   shutdown
!
interface Ethernet6
   shutdown
!
interface Ethernet7
   shutdown
!
interface Ethernet8
   shutdown
!
interface Loopback0
   ip address 10.0.1.1/32
!
interface Management1
!
ip routing
!
route-map 234frwe permit 10
!
peer-filter PF_LEAFS
   10 match as-range 65001-65109 result accept
!
router bgp 65101
   router-id 10.0.1.1
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   bgp listen range 10.2.0.0/16 peer-group LEAFS peer-filter PF_LEAFS
   neighbor LEAFS peer group
   neighbor LEAFS bfd
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES bfd
   neighbor 10.2.1.0 peer group SPINES
   neighbor 10.2.2.0 peer group SPINES
   !
   address-family ipv4
      neighbor LEAFS activate
      neighbor SPINES activate
      network 10.0.1.1/32
      redistribute connected
!
end
```
le-2
```
! Command: show running-config
! device: le-2 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
alias c configure terminal
alias ii show interfaces descr
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname le-2
!
spanning-tree mode mstp
!
interface Ethernet1
   description 2@sp-1
   mtu 8000
   no switchport
   ip address 10.2.1.3/31
   no ip ospf neighbor bfd
!
interface Ethernet2
   description 2@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.3/31
   no ip ospf neighbor bfd
!
interface Ethernet3
   description eth0@cl-2
   no ip ospf neighbor bfd
!
interface Ethernet4
   shutdown
!
interface Ethernet5
   shutdown
!
interface Ethernet6
   shutdown
!
interface Ethernet7
   shutdown
!
interface Ethernet8
   shutdown
!
interface Loopback0
   ip address 10.0.1.2/32
!
interface Management1
!
ip routing
!
router bgp 65102
   router-id 10.0.1.2
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES bfd
   neighbor 10.2.1.2 peer group SPINES
   neighbor 10.2.2.2 peer group SPINES
   !
   address-family ipv4
      neighbor SPINES activate
      network 10.0.1.2/32
      redistribute connected
!
end
```
le-3
```
! Command: show running-config
! device: le-3 (vEOS-lab, EOS-4.29.2F)
!
! boot system flash:/vEOS-lab.swi
!
no aaa root
!
alias c configure terminal
alias ii show interfaces descr
!
transceiver qsfp default-mode 4x10G
!
service routing protocols model ribd
!
hostname le-3
!
spanning-tree mode mstp
!
interface Ethernet1
   description 3@sp-1
   mtu 8000
   no switchport
   ip address 10.2.1.5/31
   no ip ospf neighbor bfd
!
interface Ethernet2
   description 3@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.5/31
   no ip ospf neighbor bfd
!
interface Ethernet3
   description eth0@cl-3
   no ip ospf neighbor bfd
!
interface Ethernet4
   description eth0@cl-4
!
interface Ethernet5
   shutdown
!
interface Ethernet6
   shutdown
!
interface Ethernet7
   shutdown
!
interface Ethernet8
   shutdown
!
interface Loopback0
   ip address 10.0.1.3/32
!
interface Management1
!
ip routing
!
router bgp 65103
   router-id 10.0.1.3
   timers bgp 3 9
   maximum-paths 4 ecmp 4
   neighbor SPINES peer group
   neighbor SPINES remote-as 65001
   neighbor SPINES bfd
   neighbor 10.2.1.4 peer group SPINES
   neighbor 10.2.2.4 peer group SPINES
   !
   address-family ipv4
      neighbor SPINES activate
      network 10.0.1.3/32
      redistribute connected
!
end
```