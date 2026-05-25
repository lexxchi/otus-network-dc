### Underlay. IS-IS


### Цель
Настроить IS-IS для Underlay сети


### Шаги
1. Настроите ISIS в Underlay сети, для IP связанности между всеми сетевыми устройствами.
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедитесь в наличии IP связанности между устройствами в ISIS домене

### Ход выполнения
#### 1. Используем схему из [лабораторной работы #2](../lab-02/)

Зачистим конфигурацию от OSPF на всех аристах:

```
interface Ethernet1-3
no ip ospf neighbor bfd
no ip ospf network point-to-point
no ip ospf authentication message-digest
no ip ospf area 0.0.0.0
no ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
default ip ospf neighbor bfd

no router ospf 1
interface Loopback0
no ip ospf area 0.0.0.0
```
#### 2. Зададим минимальную конфигурацию IS-IS
- ISIS process name
- net
- router-id ipv4
- is-type level-1
- log-adjacency-changes

```
sp-1#sh run | s isis
interface Ethernet1
   isis enable 49
   isis network point-to-point
interface Ethernet2
   isis enable 49
   isis network point-to-point
interface Ethernet3
   isis enable 49
   isis network point-to-point
interface Loopback0
   isis enable 49
router isis 49
   net 49.0001.0100.0000.1000.00
   router-id ipv4 10.0.1.0
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
sp-1#
```

```
sp-2#sh run | s isis
interface Ethernet1
   isis enable 49
   isis network point-to-point
interface Ethernet2
   isis enable 49
   isis network point-to-point
interface Ethernet3
   isis enable 49
   isis network point-to-point
interface Loopback0
   isis enable 49
router isis 49
   net 49.0001.0100.0000.2000.00
   router-id ipv4 10.0.2.0
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
```


В ходе выполнения столкнулся с проблемой непрохождения MTU 9000, ввиду невозможности выставить нужный MTU на бриджах хостовой машины выставил 8000 на интерфейсах спайнов

```
sp-1#sh run | sec mtu
interface Ethernet1
   mtu 8000
interface Ethernet2
   mtu 8000
interface Ethernet3
   mtu 8000
```

```
sp-2#sh run | sec mtu
interface Ethernet1
   mtu 8000
interface Ethernet2
   mtu 8000
interface Ethernet3
   mtu 8000
```

#### 3. Проверим IP связность

На спайнах поднялся ISIS:

```
sp-1#show isis 49 neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
49        default  le-1             L1   Ethernet1          P2P               UP    23          0D
49        default  le-2             L1   Ethernet2          P2P               UP    29          0D
49        default  le-3             L1   Ethernet3          P2P               UP    24          0E
sp-1#
```

```
sp-2#show isis 49 neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
49        default  le-1             L1   Ethernet1          P2P               UP    24          0E
49        default  le-2             L1   Ethernet2          P2P               UP    26          0E
49        default  le-3             L1   Ethernet3          P2P               UP    22          0F
sp-2#
```

Проверка на лифах ip связности:

```
le-1#show ip route

VRF: default
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

Gateway of last resort is not set

 I L1     10.0.1.0/32 [115/20] via 10.2.1.0, Ethernet1
 C        10.0.1.1/32 is directly connected, Loopback0
 I L1     10.0.1.2/32 [115/30] via 10.2.1.0, Ethernet1
                               via 10.2.2.0, Ethernet2
 I L1     10.0.2.0/32 [115/20] via 10.2.2.0, Ethernet2
 C        10.2.1.0/31 is directly connected, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.1.0, Ethernet1
 C        10.2.2.0/31 is directly connected, Ethernet2
 I L1     10.2.2.2/31 [115/20] via 10.2.2.0, Ethernet2

le-1#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=64 time=6.66 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=64 time=4.88 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=64 time=5.60 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=64 time=4.65 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=64 time=7.31 ms
```

```
le-2#show ip route

VRF: default
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

Gateway of last resort is not set

 I L1     10.0.1.0/32 [115/20] via 10.2.1.2, Ethernet1
 I L1     10.0.1.1/32 [115/30] via 10.2.1.2, Ethernet1
                               via 10.2.2.2, Ethernet2
 C        10.0.1.2/32 is directly connected, Loopback0
 I L1     10.0.2.0/32 [115/20] via 10.2.2.2, Ethernet2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.2, Ethernet1
 C        10.2.1.2/31 is directly connected, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.2, Ethernet2
 C        10.2.2.2/31 is directly connected, Ethernet2

le-2#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=64 time=5.57 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=64 time=4.33 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=64 time=5.82 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=64 time=4.36 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=64 time=4.98 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 33ms
rtt min/avg/max/mdev = 4.330/5.013/5.822/0.615 ms, ipg/ewma 8.265/5.286 ms
le-2#
```

```
le-3#show ip route

VRF: default
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

Gateway of last resort is not set

 I L1     10.0.1.0/32 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.0.1.1/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 I L1     10.0.1.2/32 [115/30] via 10.2.1.4, Ethernet1
                               via 10.2.2.4, Ethernet2
 C        10.0.1.3/32 is directly connected, Loopback0
 I L1     10.0.2.0/32 [115/20] via 10.2.2.4, Ethernet2
 I L1     10.2.1.0/31 [115/20] via 10.2.1.4, Ethernet1
 I L1     10.2.1.2/31 [115/20] via 10.2.1.4, Ethernet1
 C        10.2.1.4/31 is directly connected, Ethernet1
 I L1     10.2.2.0/31 [115/20] via 10.2.2.4, Ethernet2
 I L1     10.2.2.2/31 [115/20] via 10.2.2.4, Ethernet2
 C        10.2.2.4/31 is directly connected, Ethernet2

le-3#
le-3#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=64 time=5.15 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=64 time=4.97 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=64 time=5.30 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=64 time=5.11 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=64 time=5.50 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 23ms
rtt min/avg/max/mdev = 4.972/5.209/5.504/0.181 ms, ipg/ewma 5.752/5.190 ms
le-3#
```
#### 4. Применим рекомендации
 - Применим BFD
```
interface Ethernet1-3
   bfd echo
   isis bfd
```

Проверим bfd на спайнах:


```
sp-1#show bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.1.1  1269593610   189433519        Ethernet1(13)  normal   05/12/26 20:50
10.2.1.3  1336653791  1628458226        Ethernet2(14)  normal   05/12/26 20:50
10.2.1.5  3465630870   665399992        Ethernet3(15)  normal   05/12/26 21:12

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

sp-1#
```

```
sp-2#show bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.2.1  3988860974   197728242        Ethernet1(13)  normal   05/12/26 20:50
10.2.2.3  2129510715  1556763938        Ethernet2(14)  normal   05/12/26 20:50
10.2.2.5   608282953  2722612291        Ethernet3(15)  normal   05/12/26 21:01

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

sp-2#
```
Всё поднялось

 - Настроим аутентификацию

Добавим в конфигурацию спайнов
```
interface ethernet 1-3
isis authentication mode md5
isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
```

И лифов
```
interface ethernet 1-2
isis authentication mode md5
isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
```


Проверим на спайнах, что счётчики Hold time обновляются и что сессии не упали:
```
sp-1#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
49        default  le-1             L1   Ethernet1          P2P               UP    25          0D
49        default  le-2             L1   Ethernet2          P2P               UP    26          0D
49        default  le-3             L1   Ethernet3          P2P               UP    28          0E
sp-1#
```

```
sp-2#show isis neighbors

Instance  VRF      System Id        Type Interface          SNPA              State Hold time   Circuit Id
49        default  le-1             L1   Ethernet1          P2P               UP    24          0E
49        default  le-2             L1   Ethernet2          P2P               UP    24          0E
49        default  le-3             L1   Ethernet3          P2P               UP    28          0F
sp-2#
```


### Листинг конфигов устроств

#### sp-1
```
sp-1#sh run | no-more
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
vlan 1001
!
interface Ethernet1
   description 1@le-1
   mtu 8000
   no switchport
   ip address 10.2.1.0/31
   bfd echo
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet2
   description 1@le-2
   mtu 8000
   no switchport
   ip address 10.2.1.2/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet3
   description 1@le-3
   mtu 8000
   no switchport
   ip address 10.2.1.4/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
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
   isis enable 49
!
interface Management1
!
ip routing
!
router isis 49
   net 49.0001.0100.0000.1000.00
   router-id ipv4 10.0.1.0
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
!
end
sp-1#
```

#### sp-2
```
sp-2#sh run | no-more
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
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet2
   description 2@le-2
   mtu 8000
   no switchport
   ip address 10.2.2.2/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet3
   description 2@le-3
   mtu 8000
   no switchport
   ip address 10.2.2.4/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
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
   isis enable 49
!
interface Management1
!
ip routing
!
router isis 49
   net 49.0001.0100.0000.2000.00
   router-id ipv4 10.0.2.0
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
!
end
sp-2#
```

#### le-1
```
le-1#sh run | no-more
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
   bfd echo
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet2
   description 1@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.1/31
   bfd echo
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
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
   isis enable 49
!
interface Management1
!
ip routing
!
route-map 234frwe permit 10
!
router isis 49
   net 49.0001.0100.0000.1001.00
   router-id ipv4 10.0.1.1
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
!
end
le-1#
```

#### le-2
```
le-2#sh run | no-more
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
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet2
   description 2@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.3/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
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
   isis enable 49
!
interface Management1
!
ip routing
!
router isis 49
   net 49.0001.0100.0000.1002.00
   router-id ipv4 10.0.1.2
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
!
end
le-2#
```

#### le-3
```
le-3#sh run | no-more
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
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
!
interface Ethernet2
   description 3@sp-2
   mtu 8000
   no switchport
   ip address 10.2.2.5/31
   bfd echo
   no ip ospf neighbor bfd
   isis enable 49
   isis bfd
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 Dt3O7F8H5UZpWGToc060OuaErZJx9gMJ
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
   isis enable 49
!
interface Management1
!
ip routing
!
router isis 49
   net 49.0001.0100.0000.1003.00
   router-id ipv4 10.0.1.3
   is-type level-1
   log-adjacency-changes
   !
   address-family ipv4 unicast
!
end
le-3#
```