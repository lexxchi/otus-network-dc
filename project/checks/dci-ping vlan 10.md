# Проверка DCI для VLAN 10

## Что проверяем

Проверяем первый tenant-сценарий через DCI: VLAN 10 из `TENANT-1` растянут из DC1 в DC2 как L2-сервис через L2VNI 10010.

На этом этапе в DC2 для `TENANT-1` не настроен L3VNI и не поднят локальный SVI/anycast gateway. Клиент `cl-44` в DC2 использует gateway `192.168.10.1`, который находится в DC1. Поэтому проверка `show ip route vrf TENANT-1` на DC2 здесь не является основной: маршрутизация выполняется в DC1 после доставки кадров по L2VNI.

Проверяем:

- `cl-44` получает ARP-запись gateway `192.168.10.1` с MAC `00:00:be:ef:ca:fe`.
- `cl-44` пингует клиента DC1 в той же VLAN 10.
- `cl-44` выходит в интернет через centralized gateway/edge в DC1.

Ожидаемый путь до интернета:

```text
cl-44 -> L2VNI 10010 через DCI -> DC1 anycast gateway -> TENANT-1 -> bl-1 -> vyos-fw -> NAT -> vyos-isp
```

## Выводы команд

Проверяем ARP на `cl-44`. Видно, что gateway `192.168.10.1` доступен через anycast gateway MAC из DC1.

```text
cl-44> show arp

00:00:be:ef:ca:fe  192.168.10.1 expires in 119 seconds
```

Проверяем связность `cl-44` с клиентом DC1 в VLAN 10.

```text
cl-44> ping 192.168.10.12

84 bytes from 192.168.10.12 icmp_seq=1 ttl=64 time=63.514 ms
84 bytes from 192.168.10.12 icmp_seq=2 ttl=64 time=129.879 ms
84 bytes from 192.168.10.12 icmp_seq=3 ttl=64 time=79.208 ms
84 bytes from 192.168.10.12 icmp_seq=4 ttl=64 time=144.880 ms
84 bytes from 192.168.10.12 icmp_seq=5 ttl=64 time=63.906 ms
```

Проверяем выход `cl-44` в интернет через DC1.

```text
cl-44> ping 8.8.8.8

84 bytes from 8.8.8.8 icmp_seq=1 ttl=62 time=100.558 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=62 time=73.652 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=62 time=88.912 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=62 time=103.157 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=62 time=78.308 ms

cl-44>
```

## Вывод

VLAN 10 успешно растянут между DC1 и DC2 через EVPN DCI как L2-сервис. Клиент `cl-44` физически находится в DC2, но использует gateway и внешний выход DC1. Это подтверждает сценарий centralized gateway / centralized internet breakout через DC1.
