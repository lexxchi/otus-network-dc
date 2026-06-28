# Проверка arp и ping

## Что проверяем

Проверяем L2/L3-связность внутри fabric и доступ idrac-клиентов до имитации внешнего интернета.

## Выводы команд

Проверка на бордер лифе

```
bl-1#sh arp vrf ?
  WORD  VRF name
  all   All Virtual Routing and Forwarding instances

bl-1#sh arp vrf IDRAC
Address         Age (sec)  Hardware Addr   Interface
198.51.100.130    0:06:39  0050.7966.6812  Vlan30, Ethernet4
198.51.100.143    0:00:24  0050.7966.6809  Vlan30, Vxlan1
```

На клиенте cl-idrac-1
```
cl-idrac-1> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=62 time=21.346 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=14.069 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=62 time=18.935 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=62 time=22.494 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=62 time=15.958 ms

cl-idrac-1> tracer 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   198.51.100.129   9.926 ms  8.193 ms  8.497 ms
 2   203.0.113.6   17.420 ms  15.553 ms  13.593 ms
 3   *8.8.8.8   13.968 ms (ICMP type:3, code:3, Destination port unreachable)

cl-idrac-1>
```

На клиенте cl-idrac-2
```
cl-idrac-2> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=62 time=32.269 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=60.885 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=62 time=32.704 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=62 time=28.921 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=62 time=28.196 ms

cl-idrac-2> tracer 8.8.8.8
trace to 8.8.8.8, 8 hops max, press Ctrl+C to stop
 1   198.51.100.129   20.887 ms  22.307 ms  23.158 ms
 2   203.0.113.6   33.908 ms  26.177 ms  29.868 ms
 3   *8.8.8.8   36.997 ms (ICMP type:3, code:3, Destination port unreachable)

cl-idrac-2>
```

## Что проверяем

Проверяем что ip адреса iDRAC доступны из клиентов вланов 10 и 20, но не маршрутизацией внутри фабрики, а в интеренете

Проверка пингом

```
vyos@cl-3344:~$ sudo ping -c 4 198.51.100.143
PING 198.51.100.143 (198.51.100.143) 56(84) bytes of data.
64 bytes from 198.51.100.143: icmp_seq=1 ttl=58 time=59.6 ms
64 bytes from 198.51.100.143: icmp_seq=2 ttl=58 time=60.8 ms
64 bytes from 198.51.100.143: icmp_seq=3 ttl=58 time=86.8 ms
64 bytes from 198.51.100.143: icmp_seq=4 ttl=58 time=68.1 ms

--- 198.51.100.143 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 59.629/68.834/86.803/10.863 ms
```
Трассировка с клиента cl-3344
```
vyos@cl-3344:~$ traceroute  198.51.100.143
traceroute to 198.51.100.143 (198.51.100.143), 30 hops max, 60 byte packets
 1  192.168.20.1 (192.168.20.1)  22.510 ms  25.932 ms  34.009 ms
 2  192.168.20.1 (192.168.20.1)  73.443 ms  79.339 ms  203.264 ms
 3  203.0.113.2 (203.0.113.2)  216.765 ms  284.354 ms  287.562 ms
 4  198.51.100.2 (198.51.100.2)  355.454 ms  148.559 ms  199.932 ms
 5  * * *
 6  203.0.113.5 (203.0.113.5)  452.530 ms  293.956 ms  511.527 ms
 7  198.51.100.143 (198.51.100.143)  597.323 ms  574.601 ms  571.955 ms
 vyos@cl-3344:~$
 ```

## Что проверяем
На роутере эмуляции интернета пинги айдрака:

 ```
 vyos@vyos-isp:/$ sudo ping -c 4 198.51.100.143
PING 198.51.100.143 (198.51.100.143) 56(84) bytes of data.
64 bytes from 198.51.100.143: icmp_seq=1 ttl=62 time=30.1 ms
64 bytes from 198.51.100.143: icmp_seq=2 ttl=62 time=31.3 ms
64 bytes from 198.51.100.143: icmp_seq=3 ttl=62 time=31.8 ms
64 bytes from 198.51.100.143: icmp_seq=4 ttl=62 time=41.0 ms

--- 198.51.100.143 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 30.054/33.555/40.992/4.342 ms
vyos@vyos-isp:/$
 ```

## Что проверяем
Как разворачивается трафик в "интернете":

 ```
 vyos@vyos-isp:/$  sudo tcpdump -ani eth0 host 198.51.100.143
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
22:13:47.523131 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 1, length 64
22:13:47.523159 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 1, length 64
22:13:47.553283 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 1, length 64
22:13:47.553308 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 1, length 64
22:13:48.520258 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 2, length 64
22:13:48.520285 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 2, length 64
22:13:48.554396 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 2, length 64
22:13:48.554424 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 2, length 64
22:13:49.522267 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 3, length 64
22:13:49.522293 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 3, length 64
22:13:49.558945 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 3, length 64
22:13:49.558969 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 3, length 64
22:13:50.521731 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 4, length 64
22:13:50.521761 IP 192.0.2.20 > 198.51.100.143: ICMP echo request, id 7089, seq 4, length 64
22:13:50.550530 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 4, length 64
22:13:50.550560 IP 198.51.100.143 > 192.0.2.20: ICMP echo reply, id 7089, seq 4, length 64
```