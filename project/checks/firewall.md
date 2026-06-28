# Проверка firewall

## Что проверяем

Проверяем, что с внешней стороны доступен BGP к vyos-fw и недоступен SSH на внешний интерфейс.

## Выводы команд

```text
vyos@vyos-isp:/var/www/html$ nc -vnz 198.51.100.1 179
Connection to 198.51.100.1 179 port [tcp/*] succeeded!
vyos@vyos-isp:/var/www/html$ nc -vnz 198.51.100.1 22
nc: connect to 198.51.100.1 port 22 (tcp) failed: Connection timed out
vyos@vyos-isp:/var/www/html$
vyos@vyos-isp:/var/www/html$
```
