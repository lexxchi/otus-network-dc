### VxLAN L2 VNI

### Цель
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

### Шаги
1. Настроите BGP peering между Leaf и Spine в AF l2vpn evpn
2. Настроите связанность между клиентами в первой зоне и убедитесь в её наличии
3. Зафиксируете в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

### Ход выполнения

#### 0. Бриф по настройке
   1. Включаем multi-agent
   2. Настроем family evpn
   3. Настроем интерфейс vxlan


#### 1. Используем схему из [лабораторной работы #5](../lab-05/)

Для андерлея у нас уже всё готово из прошлой лабы, поэтому нам достаточно будет добавить address family evpn.

При попытке получаем ошибку:
```
sp-1(config)#router bgp 65001
sp-1(config-router-bgp)#
sp-1(config-router-bgp)#   address-family evpn
! Routing protocols model multi-agent must be configured for EVPN address-family
```
Нужно включить multi-agent, для этого необходимо дать команду и перезагрузить устройтсво. Выполним на всех устройствах
```
service routing protocols model multi-agent
```

#### 2. Зададим минимальную конфигурацию eBGP address family evpn

```
sp-1(config)#router bgp 65001
sp-1(config-router-bgp)#
sp-1(config-router-bgp)#address-family evpn
sp-1(config-router-bgp-af)#neighbor LEAFS activate
```

#### 3. Настроем интерфейс vxlan

```
le-1#c
le-1(config)#interface vxlan1
le-1(config-if-Vx1)#vxlan udp-port 4789
le-1(config-if-Vx1)#vxlan vlan 10 vni 10010
le-1(config-if-Vx1)#vxlan vlan 20 vni 10020
le-1(config-if-Vx1)#vxlan source-interface loopback 0
le-1(config-if-Vx1)#end
le-1#
```

#### 4. Пробросим вланы в evpn
```
router bgp 65101
   vlan 10
      rd 10.0.1.1:10010
      route-target both 10:10010
      redistribute learned
```

#### 5. Настроем ip клиенты

Для минимальной проверки настроем влан 10 аксессом на le-1 и le-2:
```
le-1#sh run interfaces et3
interface Ethernet3
   description eth0@cl-1
   switchport access vlan 10
```

```
le-2#sh run interfaces et3
interface Ethernet3
   description eth0@cl-2
   switchport access vlan 10
```

А так же IP на самихз клиентах (были настроены ранее, когда я проверял static vtep):

```
cl-1> show ip

NAME        : cl-1[1]
IP/MASK     : 1.1.1.1/24
GATEWAY     : 1.1.1.254
DNS         :
MAC         : 00:50:79:66:68:07
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

cl-1>
```

```
cl-2> show ip

NAME        : cl-2[1]
IP/MASK     : 1.1.1.2/24
GATEWAY     : 1.1.1.254
DNS         :
MAC         : 00:50:79:66:68:08
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
```

Попробуем пинги:

```
cl-1> ping 1.1.1.2
host (1.1.1.2) not reachable
```

```
cl-2> ping 1.1.1.1
host (1.1.1.1) not reachable
```

#### 6. Проверим evpn связность и наличие маршрутов type-2 и type-3:

На спайне
```
sp-2#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.2.0, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd Pfc
  10.2.2.1 4 65101           1653      1659    0    0 01:09:51 Estab   2      2
  10.2.2.3 4 65102            650       644    0    0 00:26:59 Estab   2      2
  10.2.2.5 4 65103            632       632    0    0 00:26:20 Estab   0      0
```

На лифе
```
le-2#show bgp evpn su
BGP summary information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 65001            905       899    0    0 00:27:53 Estab   2      2
  10.2.2.2 4 65001            901       913    0    0 00:27:53 Estab   2      2
le-2#show bgp evpn
BGP routing table information for VRF default
Router identifier 10.0.1.2, local AS number 65102
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.1:10010 mac-ip 0050.7966.6807
                                 10.0.1.1              -       100     0       65001 65101 i
 *  ec    RD: 10.0.1.1:10010 mac-ip 0050.7966.6807
                                 10.0.1.1              -       100     0       65001 65101 i
 * >      RD: 10.0.1.2:10010 mac-ip 0050.7966.6808
                                 -                     -       -       0       i
 * >Ec    RD: 10.0.1.1:10010 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65001 65101 i
 *  ec    RD: 10.0.1.1:10010 imet 10.0.1.1
                                 10.0.1.1              -       100     0       65001 65101 i
 * >      RD: 10.0.1.2:10010 imet 10.0.1.2
                                 -                     -       -       0       i
le-2#
```
Кажется что сам evpn работает, но чего-то не хватает.

Судя по выводу на удалённых маршрутах нет RT, хотя он есть на локальных
```
le-1#show bgp evpn route-type mac-ip detail
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
BGP routing table entry for mac-ip 0050.7966.6807, Route Distinguisher: 10.0.1.1:10010
 Paths: 1 available
  Local
    - from - (0.0.0.0)
      Origin IGP, metric -, localpref -, weight 0, tag 0, valid, local, best
      Extended Community: Route-Target-AS:10:10010 TunnelEncap:tunnelTypeVxlan
      VNI: 10010 ESI: 0000:0000:0000:0000:0000
BGP routing table entry for mac-ip 0050.7966.6808, Route Distinguisher: 10.0.1.2:10010
 Paths: 2 available
  65001 65102
    10.0.1.2 from 10.2.1.0 (10.0.1.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP head, ECMP, best, ECMP contributor
      VNI: 10010 ESI: 0000:0000:0000:0000:0000
  65001 65102
    10.0.1.2 from 10.2.2.0 (10.0.2.0)
      Origin IGP, metric -, localpref 100, weight 0, tag 0, valid, external, ECMP, ECMP contributor
      VNI: 10010 ESI: 0000:0000:0000:0000:0000
```

Добавим send-community extended на spine и leaf:

```
le-2(config)#router bgp 65102
le-2(config-router-bgp)#   neighbor SPINES send-community extended
le-2(config-router-bgp)#end
```

Теперь устройства увидени друг друга

```
cl-1> ping 1.1.1.2

84 bytes from 1.1.1.2 icmp_seq=1 ttl=64 time=21.390 ms
84 bytes from 1.1.1.2 icmp_seq=2 ttl=64 time=23.632 ms
84 bytes from 1.1.1.2 icmp_seq=3 ttl=64 time=18.413 ms
84 bytes from 1.1.1.2 icmp_seq=4 ttl=64 time=38.431 ms
84 bytes from 1.1.1.2 icmp_seq=5 ttl=64 time=18.832 ms

cl-1>
```


```
cl-2> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=64 time=25.691 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=64 time=22.419 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=64 time=23.552 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=64 time=22.853 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=64 time=19.437 ms

cl-2>
```

#### 7. Добавим ещё клиент

Добавим влан 10 на le-3
```
interface Ethernet3
   switchport access vlan 10
```
Пропишем адрес и проверим что есть пинг до двух других клиентов во влане 10

```
cl-3> set pcname cl-3
cl-3> ip 1.1.1.3/24 1.1.1.254
cl-3>
cl-3> ping 1.1.1.2

84 bytes from 1.1.1.2 icmp_seq=1 ttl=64 time=18.931 ms
84 bytes from 1.1.1.2 icmp_seq=2 ttl=64 time=24.550 ms
84 bytes from 1.1.1.2 icmp_seq=3 ttl=64 time=19.974 ms
84 bytes from 1.1.1.2 icmp_seq=4 ttl=64 time=20.957 ms
84 bytes from 1.1.1.2 icmp_seq=5 ttl=64 time=19.187 ms
cl-3> ping 1.1.1.1

84 bytes from 1.1.1.1 icmp_seq=1 ttl=64 time=18.065 ms
84 bytes from 1.1.1.1 icmp_seq=2 ttl=64 time=17.603 ms
84 bytes from 1.1.1.1 icmp_seq=3 ttl=64 time=21.873 ms
84 bytes from 1.1.1.1 icmp_seq=4 ttl=64 time=18.394 ms
84 bytes from 1.1.1.1 icmp_seq=5 ttl=64 time=20.183 ms
```

#### 8. Запишем итоговую таблицу

| Leaf |       RD       |
|------|--------------- |
| le-1 | 10.0.1.1:10010 |
| le-2 | 10.0.1.2:10010 |
| le-3 | 10.0.1.3:10010 |

| VLAN |    RT    |
|------|----------|
|  10  | 10:10010 |
|  20  | 20:10020 |

### Листинг конфигов устройств

Перечислен в папке [configs] (./configs/)

#### TODO
- Добавить влан 20 и 2 клиента в нём
- Переделать на vxlan aware схему
- Вывести spine в maintenance
