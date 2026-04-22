### Underlay. OSPF

### Цель
Настроить OSPF для Underlay сети


### Шаги
1. Настроите OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
2. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедитесь в наличии IP связанности между устройствами в OSFP домене

### Ход выполнения
#### 1. Используем схему из [лабораторной работы #1](lab-01/)
#### 2. Добавим Lo и настроим ospf
- Согласно IP плану распределим ip адреса

|Device|Interface|IP Address/Subnet Mask
|---|---|---|
sp-1|Loopback0|10.0.1.0/32
sp-2|Loopback0|10.0.2.0/32
le-1|Loopback0|10.0.1.1/32
le-2|Loopback0|10.0.1.2/32
le-3|Loopback0|10.0.1.3/32

- Пропишем на Lo0 всех арист
```
interface Loopback0
   ip address 10.0.1.0/32
   ip ospf area 0.0.0.0
```
- Включаем на всех аристах  **ip routing**
- Настраиваем ospf
```
router ospf 1
   router-id 10.0.1.0
```

- выставляем настройки на ospf интерфейсах:
```
   mtu 9214
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
```
- Проверяем наличие нейборов и доступность лупбеков
```
sp-1#show ip ospf nei
Neighbor ID     Instance VRF      Pri State                  Dead Time   Address         Interface
10.0.1.1        1        default  0   FULL                   00:00:38    10.2.1.1        Ethernet1
10.0.1.2        1        default  0   FULL                   00:00:30    10.2.1.3        Ethernet2
10.0.1.3        1        default  0   FULL                   00:00:30    10.2.1.5        Ethernet3

sp-1#show ip route ospf

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

 O        10.0.1.1/32 [110/20] via 10.2.1.1, Ethernet1
 O        10.0.1.2/32 [110/20] via 10.2.1.3, Ethernet2
 O        10.0.1.3/32 [110/20] via 10.2.1.5, Ethernet3
 O        10.0.2.0/32 [110/30] via 10.2.1.1, Ethernet1
                               via 10.2.1.3, Ethernet2
                               via 10.2.1.5, Ethernet3
 O        10.2.2.0/31 [110/20] via 10.2.1.1, Ethernet1
 O        10.2.2.2/31 [110/20] via 10.2.1.3, Ethernet2
 O        10.2.2.4/31 [110/20] via 10.2.1.5, Ethernet3

sp-1#
sp-1#ping 10.0.1.1
PING 10.0.1.1 (10.0.1.1) 72(100) bytes of data.
80 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=9.38 ms
80 bytes from 10.0.1.1: icmp_seq=2 ttl=64 time=4.13 ms
80 bytes from 10.0.1.1: icmp_seq=3 ttl=64 time=3.56 ms
80 bytes from 10.0.1.1: icmp_seq=4 ttl=64 time=3.64 ms
80 bytes from 10.0.1.1: icmp_seq=5 ttl=64 time=4.03 ms

--- 10.0.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 39ms
rtt min/avg/max/mdev = 3.560/4.951/9.382/2.226 ms, ipg/ewma 9.810/7.089 ms
sp-1#ping 10.0.1.2
PING 10.0.1.2 (10.0.1.2) 72(100) bytes of data.
80 bytes from 10.0.1.2: icmp_seq=1 ttl=64 time=7.48 ms
80 bytes from 10.0.1.2: icmp_seq=2 ttl=64 time=4.56 ms
80 bytes from 10.0.1.2: icmp_seq=3 ttl=64 time=4.75 ms
80 bytes from 10.0.1.2: icmp_seq=4 ttl=64 time=4.79 ms
80 bytes from 10.0.1.2: icmp_seq=5 ttl=64 time=7.03 ms

--- 10.0.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 35ms
rtt min/avg/max/mdev = 4.569/5.725/7.487/1.264 ms, ipg/ewma 8.813/6.629 ms
sp-1#ping 10.0.1.3
PING 10.0.1.3 (10.0.1.3) 72(100) bytes of data.
80 bytes from 10.0.1.3: icmp_seq=1 ttl=64 time=6.92 ms
80 bytes from 10.0.1.3: icmp_seq=2 ttl=64 time=4.57 ms
80 bytes from 10.0.1.3: icmp_seq=3 ttl=64 time=5.16 ms
80 bytes from 10.0.1.3: icmp_seq=4 ttl=64 time=4.87 ms
80 bytes from 10.0.1.3: icmp_seq=5 ttl=64 time=3.60 ms

--- 10.0.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 36ms
rtt min/avg/max/mdev = 3.609/5.030/6.928/1.085 ms, ipg/ewma 9.229/5.923 ms
sp-1#ping 10.0.2.0
PING 10.0.2.0 (10.0.2.0) 72(100) bytes of data.
80 bytes from 10.0.2.0: icmp_seq=1 ttl=63 time=12.5 ms
80 bytes from 10.0.2.0: icmp_seq=2 ttl=63 time=8.30 ms
80 bytes from 10.0.2.0: icmp_seq=3 ttl=63 time=8.58 ms
80 bytes from 10.0.2.0: icmp_seq=4 ttl=63 time=9.50 ms
80 bytes from 10.0.2.0: icmp_seq=5 ttl=63 time=8.61 ms

--- 10.0.2.0 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 52ms
rtt min/avg/max/mdev = 8.304/9.506/12.522/1.564 ms, ipg/ewma 13.137/10.974 ms
sp-1#
```

Делаем вывод - ospf работает, spine1 видит lo spine2

#### 3. Применим рекомендации
 - Применим BFD и настроим аутентификацию
```
interface Ethernet1-3
   ip ospf authentication message-digest
   ip ospf message-digest-key 1 md5 ospf-laba
```

```
interface Ethernet1-3
   ip ospf neighbor bfd
   bfd echo
```

- Проверка

```
sp-1#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.1.1  1156622677  3412472140        Ethernet1(13)  normal   04/21/26 19:40
10.2.1.3  3657135524  1026421788        Ethernet2(14)  normal   04/21/26 19:40
10.2.1.5  1035464494   862279889        Ethernet3(15)  normal   04/21/26 19:38

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

sp-1#
```

```
sp-2#sh bfd peers
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp
--------- ----------- ----------- -------------------- ------- ----------------
10.2.2.1  3144467371  2149913633        Ethernet1(13)  normal   04/21/26 19:40
10.2.2.3  2681390717  1283612306        Ethernet2(14)  normal   04/21/26 19:40
10.2.2.5  1103508868   224692985        Ethernet3(15)  normal   04/21/26 19:39

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

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
   mtu 9214
   no switchport
   ip address 10.2.1.0/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
!
interface Ethernet2
   description 1@le-2
   mtu 9214
   no switchport
   ip address 10.2.1.2/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
!
interface Ethernet3
   description 1@le-3
   mtu 9214
   no switchport
   ip address 10.2.1.4/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
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
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 1
   router-id 10.0.1.0
   max-lsa 12000
!
end
```

#### sp-2

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
   mtu 9214
   no switchport
   ip address 10.2.2.0/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
!
interface Ethernet2
   description 2@le-2
   mtu 9214
   no switchport
   ip address 10.2.2.2/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
!
interface Ethernet3
   description 2@le-3
   mtu 9214
   no switchport
   ip address 10.2.2.4/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
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
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 1
   router-id 10.0.2.0
   max-lsa 12000
!
end
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
username cam privilege 15 secret sha512 $6$4yzDxbKRdVgDWxgZ$I3sSbpAwrADog6IjHCi6bFk0QjZzT2sQwGbE2RgmPdgTu.mzEIg8pwyw4N4dSaeRhreWDs2/mE6svAewd/PK..
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
vlan 1001
!
aaa authorization exec default local
!
interface Ethernet1
   description 1@sp-1
   mtu 9214
   no switchport
   ip address 10.2.1.1/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
!
interface Ethernet2
   description 1@sp-2
   mtu 9214
   no switchport
   ip address 10.2.2.1/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
!
interface Ethernet3
   description eth0@cl-1
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
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 1
   router-id 10.0.1.1
   max-lsa 12000
!
end
```

#### le-2

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
   mtu 9214
   no switchport
   ip address 10.2.1.3/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
!
interface Ethernet2
   description 2@sp-2
   mtu 9214
   no switchport
   ip address 10.2.2.3/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
!
interface Ethernet3
   description eth0@cl-2
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
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 1
   router-id 10.0.1.2
   max-lsa 12000
!
end
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
   mtu 9214
   no switchport
   ip address 10.2.1.5/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 aitZFw6f8+UGnpX0gG1VPA==
!
interface Ethernet2
   description 3@sp-2
   mtu 9214
   no switchport
   ip address 10.2.2.5/31
   bfd echo
   ip ospf neighbor bfd
   ip ospf network point-to-point
   ip ospf authentication message-digest
   ip ospf area 0.0.0.0
   ip ospf message-digest-key 1 md5 7 GIbvj9/hBHm4Dip6lJcyzA==
!
interface Ethernet3
   description eth0@cl-3
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
   ip ospf area 0.0.0.0
!
interface Management1
!
ip routing
!
router ospf 1
   router-id 10.0.1.3
   max-lsa 12000
!
end
```