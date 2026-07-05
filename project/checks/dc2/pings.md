# Проверка ping

## Что проверяем

Проверяем L2/L3-связность внутри автономного DC2: доступ клиента до anycast gateway VLAN 210 и связность между `cl-dc2-1`, `cl-dc2-2` и `cl-dc2-3`.

## Выводы команд

```
cl-dc2-1> ping 192.168.210.12

84 bytes from 192.168.210.12 icmp_seq=1 ttl=64 time=22.432 ms
84 bytes from 192.168.210.12 icmp_seq=2 ttl=64 time=90.098 ms
84 bytes from 192.168.210.12 icmp_seq=3 ttl=64 time=23.989 ms
84 bytes from 192.168.210.12 icmp_seq=4 ttl=64 time=22.526 ms
84 bytes from 192.168.210.12 icmp_seq=5 ttl=64 time=20.479 ms

cl-dc2-1>
```


```
cl-dc2-2> ping 192.168.210.1

84 bytes from 192.168.210.1 icmp_seq=1 ttl=64 time=3.757 ms
84 bytes from 192.168.210.1 icmp_seq=2 ttl=64 time=3.810 ms
84 bytes from 192.168.210.1 icmp_seq=3 ttl=64 time=12.105 ms
84 bytes from 192.168.210.1 icmp_seq=4 ttl=64 time=4.359 ms
84 bytes from 192.168.210.1 icmp_seq=5 ttl=64 time=5.086 ms

cl-dc2-2> ping 192.168.210.11

84 bytes from 192.168.210.11 icmp_seq=1 ttl=64 time=24.402 ms
84 bytes from 192.168.210.11 icmp_seq=2 ttl=64 time=24.064 ms
84 bytes from 192.168.210.11 icmp_seq=3 ttl=64 time=82.135 ms
84 bytes from 192.168.210.11 icmp_seq=4 ttl=64 time=23.189 ms
84 bytes from 192.168.210.11 icmp_seq=5 ttl=64 time=78.634 ms

cl-dc2-2>
```

```
cl-dc2-3> ping 192.168.210.11

84 bytes from 192.168.210.11 icmp_seq=1 ttl=64 time=20.583 ms
84 bytes from 192.168.210.11 icmp_seq=2 ttl=64 time=21.324 ms
84 bytes from 192.168.210.11 icmp_seq=3 ttl=64 time=22.707 ms
84 bytes from 192.168.210.11 icmp_seq=4 ttl=64 time=24.932 ms
84 bytes from 192.168.210.11 icmp_seq=5 ttl=64 time=27.408 ms

cl-dc2-3>
```
