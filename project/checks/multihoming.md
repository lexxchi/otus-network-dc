# Проверка EVPN multihoming

## Что проверяем

Проверяем dual-homed подключение клиента `cl-3344` к leaf-коммутаторам `le-3` и `le-4` через EVPN multihoming / ESI-LAG.

Проверки включают:

- состояние access-интерфейсов и `Port-Channel5` на `le-3`/`le-4`
- работу `link tracking group CORE-TRACKING`
- реакцию link tracking при потере uplink в сторону spine
- наличие EVPN route-type 1 auto-discovery routes
- наличие EVPN route-type 4 ethernet-segment routes
- состояние LACP bond на клиенте `cl-3344`
- доступ клиента `cl-3344` во внешний сегмент через `vyos-fw` и NAT
- прикладную проверку через `curl`, `ping` и `iperf3`

## Состояние Port-Channel на le-3

Проверяем, что физический интерфейс `Ethernet5` и агрегированный интерфейс `Port-Channel5` подняты на `le-3`.

```text
le-3#ii | i cl-3344
Et5                            up             up                 1@cl-3344
Po5                            up             up                 cl-3344
le-3#show interfaces po5
Port-Channel5 is up, line protocol is up (connected)
  Hardware is Port-Channel, address is 5001.0003.0005
  Description: cl-3344
  Ethernet MTU 9214 bytes, BW 1000000 kbit
  Full-duplex, 1Gb/s
  Active members in this channel: 1
  ... Ethernet5 , Full-duplex, 1Gb/s
  Fallback mode is: off
  Up 50 minutes, 17 seconds
  5 link status changes since last clear
  Last clearing of "show interface" counters never
  5 minutes input rate 0 bps (0.0% with framing overhead), 0 packets/sec
  5 minutes output rate 0 bps (0.0% with framing overhead), 0 packets/sec
     8959 packets input, 13085783 bytes
     Received 119 broadcasts, 223 multicast
     0 input errors, 0 input discards
     11004 packets output, 1170245 bytes
     Sent 0 broadcasts, 3871 multicast
     0 output errors, 0 output discards
le-3#
```

## Link tracking на le-3

Проверяем, что uplink-интерфейсы к spine входят в upstream-группу, а клиентский `Ethernet5` входит в downstream-группу.

```text
le-3#show link tracking group detail
Link State Group: CORE-TRACKING Status: up
Upstream Interfaces : Ethernet2 Ethernet1
Downstream Interfaces : Ethernet5
Number of times disabled : 1
Last disabled 0:53:41 ago
le-3#
```

## Состояние Port-Channel на le-4

Проверяем, что физический интерфейс `Ethernet5` и агрегированный интерфейс `Port-Channel5` подняты на `le-4`.

```text
le-4#ii | i cl-3344
Et5                            up             up                 2@cl-3344
Po5                            up             up                 cl-3344
le-4#show interfaces po5
Port-Channel5 is up, line protocol is up (connected)
  Hardware is Port-Channel, address is 5001.0010.0005
  Description: cl-3344
  Ethernet MTU 9214 bytes, BW 1000000 kbit
  Full-duplex, 1Gb/s
  Active members in this channel: 1
  ... Ethernet5 , Full-duplex, 1Gb/s
  Fallback mode is: off
  Up 53 minutes, 35 seconds
  11 link status changes since last clear
  Last clearing of "show interface" counters never
  5 minutes input rate 0 bps (0.0% with framing overhead), 0 packets/sec
  5 minutes output rate 0 bps (0.0% with framing overhead), 0 packets/sec
     7969 packets input, 5244672 bytes
     Received 2 broadcasts, 219 multicast
     0 input errors, 0 input discards
     8096 packets output, 935929 bytes
     Sent 0 broadcasts, 3806 multicast
     0 output errors, 0 output discards
le-4#
```

## Link tracking на le-4

Проверяем, что `le-4` также отслеживает состояние uplink-интерфейсов и может отключить downstream-порт к клиенту при потере связности с fabric.

```text
le-4#show link tracking group detail
Link State Group: CORE-TRACKING Status: up
Upstream Interfaces : Ethernet2 Ethernet1
Downstream Interfaces : Ethernet5
Number of times disabled : 1
Last disabled 0:57:40 ago
```

## Отработка link tracking при потере uplink

Проверяем, что при отключении uplink-интерфейсов `Ethernet1`/`Ethernet2` на `le-4` downstream-интерфейс `Ethernet5` и `Port-Channel5` также переходят в down.

```text
le-4#ii
Interface                      Status         Protocol           Description
Et1                            down           down               5@sp-1
Et2                            down           down               5@sp-2
Et3                            admin down     down
Et4                            admin down     down
Et5                            down           down               2@cl-3344
Et6                            admin down     down
Et7                            admin down     down
Et8                            admin down     down
Lo0                            up             up
Ma1                            up             up
Po5                            down           lowerlayerdown     cl-3344
Vl10                           up             up
Vl20                           up             up
Vl4094                         up             up
Vx1                            up             up
le-4#
```

## Логи link tracking

Проверяем, что в логах leaf-коммутаторов есть сообщения об отключении downstream-интерфейса из-за потери uplink.

```text
le-3#sh log | grep err
Console logging: level errors
Monitor logging: level errors
Jun 28 15:06:19 le-3 Ebra: %ETH-4-ERRDISABLE: uplink-failure-detection error detected on Ethernet5.
le-3#

le-4#sh log | grep err
Console logging: level errors
Monitor logging: level errors
Jun 28 15:02:08 le-4 Ebra: %ETH-4-ERRDISABLE: uplink-failure-detection error detected on Ethernet5.
le-4#
```

## EVPN route-type 1

Проверяем на другом leaf, что в EVPN появились auto-discovery routes для общего ESI `0000:0000:0000:0000:0001` от `le-3` и `le-4`.

```text
le-1#show bgp evpn route-type auto-discovery
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.3:10010 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10010 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.3:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.5:10010 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10010 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
 * >Ec    RD: 10.0.1.5:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:10020 auto-discovery 0 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
 * >Ec    RD: 10.0.1.3:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.5:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:1 auto-discovery 0000:0000:0000:0000:0001
                                 10.0.1.5              -       100     0       65001 65105 i
le-1#
```

## EVPN route-type 4

Проверяем на другом leaf, что в EVPN появились ethernet-segment routes для `le-3` и `le-4` с одинаковым ESI.

```text
le-1#show bgp evpn route-type ethernet-segment
BGP routing table information for VRF default
Router identifier 10.0.1.1, local AS number 65101
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.0.1.3:1 ethernet-segment 0000:0000:0000:0000:0001 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 *  ec    RD: 10.0.1.3:1 ethernet-segment 0000:0000:0000:0000:0001 10.0.1.3
                                 10.0.1.3              -       100     0       65001 65103 i
 * >Ec    RD: 10.0.1.5:1 ethernet-segment 0000:0000:0000:0000:0001 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
 *  ec    RD: 10.0.1.5:1 ethernet-segment 0000:0000:0000:0000:0001 10.0.1.5
                                 10.0.1.5              -       100     0       65001 65105 i
le-1#
```

## Состояние клиента cl-3344

Проверяем, что на клиенте поднят `bond0`, VLAN subinterface `bond0.20` имеет адрес `192.168.20.44/24`, а оба member-интерфейса находятся в up/up.

```text
vyos@cl-3344:~$ show int
Codes: S - State, L - Link, u - Up, D - Down, A - Admin Down
Interface    IP Address        MAC                VRF        MTU  S/L    Description
-----------  ----------------  -----------------  -------  -----  -----  -------------
bond0        -                 50:01:00:11:00:01  default   1500  u/u    le-1-2
bond0.20     192.168.20.44/24  50:01:00:11:00:01  default   1500  u/u
eth0         -                 50:01:00:11:00:00  default   1500  A/D
eth1         -                 50:01:00:11:00:01  default   1500  u/u    5@le-3
eth2         -                 50:01:00:11:00:01  default   1500  u/u    5@le-4
eth3         -                 50:01:00:11:00:03  default   1500  A/D
lo           127.0.0.1/8       00:00:00:00:00:00  default  65536  u/u
             ::1/128
vyos@cl-3344:~$
```

## LACP на клиенте cl-3344

Проверяем, что `bond0` работает в LACP mode `802.3ad`, использует оба member-интерфейса и видит один общий remote system-id от пары `le-3`/`le-4`.

```text
vyos@cl-3344:~$ show int bonding lacp detail
Interface    Members    Mode    Rate    System-MAC         Hash
-----------  ---------  ------  ------  -----------------  --------
bond0        eth2,eth1  active  slow    50:01:00:11:00:01  layer3+4
vyos@cl-3344:~$

vyos@cl-3344:~$ show int bonding bond0 lacp neighbors
Interface    Member    Local ID           Remote ID
-----------  --------  -----------------  -----------------
bond0        eth1      50:01:00:11:00:01  11:11:22:22:33:33
bond0        eth2      50:01:00:11:00:01  11:11:22:22:33:33
vyos@cl-3344:~$
```

## Проверка HTTP во внешний сегмент

Проверяем, что клиент `cl-3344` выходит через fabric, border leaf, `vyos-fw`, NAT и получает HTTP-ответ от имитации интернета на `8.8.8.8`.

```text
vyos@cl-3344:~$ curl 8.8.8.8
<h1>Internet</h1>
<p>Hello from simulated Internet!</p>
vyos@cl-3344:~$
```

## Проверка ICMP во внешний сегмент

Проверяем IP-связность `cl-3344` до loopback `8.8.8.8` на `vyos-isp`.

```text
vyos@cl-3344:~$ sudo ping -c 5 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=74.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=61 time=31.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=61 time=20.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=61 time=36.5 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=61 time=25.9 ms

--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 20.406/37.682/74.291/19.076 ms
vyos@cl-3344:~$
```

## Проверка iperf3

Проверяем прикладную TCP-связность и прохождение нагрузки от `cl-3344` до `8.8.8.8`.

```text
vyos@cl-3344:~$ iperf3 -c 8.8.8.8 -P 4 -b 4M -t 10
Connecting to host 8.8.8.8, port 5201
[  5] local 192.168.20.44 port 38988 connected to 8.8.8.8 port 5201
[  7] local 192.168.20.44 port 38990 connected to 8.8.8.8 port 5201
[  9] local 192.168.20.44 port 39002 connected to 8.8.8.8 port 5201
[ 11] local 192.168.20.44 port 39006 connected to 8.8.8.8 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   128 KBytes  1.05 Mbits/sec   19   4.24 KBytes
[  7]   0.00-1.00   sec   128 KBytes  1.05 Mbits/sec   20   8.48 KBytes
[  9]   0.00-1.00   sec   128 KBytes  1.05 Mbits/sec   26   11.3 KBytes
[ 11]   0.00-1.00   sec   128 KBytes  1.05 Mbits/sec   18   11.3 KBytes
[SUM]   0.00-1.00   sec   512 KBytes  4.19 Mbits/sec   83
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   1.00-2.00   sec  96.8 KBytes   793 Kbits/sec    3   17.0 KBytes
[  7]   1.00-2.00   sec   128 KBytes  1.05 Mbits/sec    2   15.6 KBytes
[  9]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   10   5.66 KBytes
[ 11]   1.00-2.00   sec  0.00 Bytes  0.00 bits/sec   10   17.0 KBytes
[SUM]   1.00-2.00   sec   225 KBytes  1.84 Mbits/sec   25
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   2.00-3.00   sec   133 KBytes  1.09 Mbits/sec    0   17.0 KBytes
[  7]   2.00-3.00   sec  84.8 KBytes   695 Kbits/sec    0   18.4 KBytes
[  9]   2.00-3.00   sec  0.00 Bytes  0.00 bits/sec    5   5.66 KBytes
[ 11]   2.00-3.00   sec  61.5 KBytes   504 Kbits/sec   12   12.7 KBytes
[SUM]   2.00-3.00   sec   279 KBytes  2.29 Mbits/sec   17
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   3.00-4.00   sec  26.2 KBytes   215 Kbits/sec    0   17.0 KBytes
[  7]   3.00-4.00   sec  0.00 Bytes  0.00 bits/sec    0   17.0 KBytes
[  9]   3.00-4.00   sec  86.9 KBytes   712 Kbits/sec    7   14.1 KBytes
[ 11]   3.00-4.00   sec  0.00 Bytes  0.00 bits/sec    0   14.1 KBytes
[SUM]   3.00-4.00   sec   113 KBytes   927 Kbits/sec    7
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   4.00-5.00   sec   101 KBytes   828 Kbits/sec    0   19.8 KBytes
[  7]   4.00-5.00   sec   127 KBytes  1.04 Mbits/sec    0   19.8 KBytes
[  9]   4.00-5.00   sec  41.1 KBytes   336 Kbits/sec    0   19.8 KBytes
[ 11]   4.00-5.00   sec  66.5 KBytes   545 Kbits/sec    0   19.8 KBytes
[SUM]   4.00-5.00   sec   336 KBytes  2.75 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   5.00-6.00   sec   127 KBytes  1.04 Mbits/sec    0   19.8 KBytes
[  7]   5.00-6.00   sec   127 KBytes  1.04 Mbits/sec    0   17.0 KBytes
[  9]   5.00-6.00   sec  96.1 KBytes   787 Kbits/sec    0   19.8 KBytes
[ 11]   5.00-6.00   sec  97.5 KBytes   799 Kbits/sec    0   22.6 KBytes
[SUM]   5.00-6.00   sec   448 KBytes  3.67 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   6.00-7.00   sec   127 KBytes  1.04 Mbits/sec    0   17.0 KBytes
[  7]   6.00-7.00   sec  0.00 Bytes  0.00 bits/sec    0   17.0 KBytes
[  9]   6.00-7.00   sec   127 KBytes  1.04 Mbits/sec    0   22.6 KBytes
[ 11]   6.00-7.00   sec   110 KBytes   904 Kbits/sec    0   22.6 KBytes
[SUM]   6.00-7.00   sec   365 KBytes  2.99 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   7.00-8.00   sec  0.00 Bytes  0.00 bits/sec    0   14.1 KBytes
[  7]   7.00-8.00   sec   127 KBytes  1.04 Mbits/sec    0   17.0 KBytes
[  9]   7.00-8.00   sec   127 KBytes  1.04 Mbits/sec    0   25.5 KBytes
[ 11]   7.00-8.00   sec  48.2 KBytes   395 Kbits/sec    0   22.6 KBytes
[SUM]   7.00-8.00   sec   303 KBytes  2.48 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   8.00-9.00   sec   127 KBytes  1.04 Mbits/sec    0   14.1 KBytes
[  7]   8.00-9.00   sec  45.4 KBytes   372 Kbits/sec    0   17.0 KBytes
[  9]   8.00-9.00   sec  33.4 KBytes   273 Kbits/sec    0   19.8 KBytes
[ 11]   8.00-9.00   sec   117 KBytes   961 Kbits/sec    0   25.5 KBytes
[SUM]   8.00-9.00   sec   323 KBytes  2.65 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]   9.00-10.00  sec  0.00 Bytes  0.00 bits/sec    0   5.66 KBytes
[  7]   9.00-10.00  sec  81.9 KBytes   670 Kbits/sec    0   5.66 KBytes
[  9]   9.00-10.00  sec  93.9 KBytes   769 Kbits/sec    0   5.66 KBytes
[ 11]   9.00-10.00  sec   110 KBytes   903 Kbits/sec    0   28.3 KBytes
[SUM]   9.00-10.00  sec   286 KBytes  2.34 Mbits/sec    0
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   867 KBytes   710 Kbits/sec   22             sender
[  5]   0.00-10.10  sec   734 KBytes   595 Kbits/sec                  receiver
[  7]   0.00-10.00  sec   850 KBytes   696 Kbits/sec   22             sender
[  7]   0.00-10.10  sec   693 KBytes   562 Kbits/sec                  receiver
[  9]   0.00-10.00  sec   734 KBytes   601 Kbits/sec   48             sender
[  9]   0.00-10.10  sec   584 KBytes   474 Kbits/sec                  receiver
[ 11]   0.00-10.00  sec   740 KBytes   606 Kbits/sec   40             sender
[ 11]   0.00-10.10  sec   576 KBytes   467 Kbits/sec                  receiver
[SUM]   0.00-10.00  sec  3.12 MBytes  2.61 Mbits/sec  132             sender
[SUM]   0.00-10.10  sec  2.53 MBytes  2.10 Mbits/sec                  receiver

iperf Done.
vyos@cl-3344:~$
```
