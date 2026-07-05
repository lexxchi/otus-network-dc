# Проверка MLAG

## Что проверяем

Проверяем состояние MLAG-пары le-1/le-2 и dual-homed клиента cl-1122.

## Выводы команд

```text
le-1#show mlag interfaces
                                                                   local/remote
  mlag       desc                state       local       remote          status
--------- ------------- ----------------- ----------- ------------ ------------
    55       cl-1122       active-full        Po55         Po55           up/up
le-1#

le-1#show mlag
MLAG Configuration:
domain-id                          :          LEAVES-1-2
local-interface                    :            Vlan4094
peer-address                       :        172.16.101.2
peer-link                          :      Port-Channel78
hb-peer-address                    :         192.168.0.2
hb-peer-vrf                        :                MGMT
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:01:00:27:03:91
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1

le-1#


le-2#show mlag interfaces
                                                                   local/remote
  mlag       desc                state       local       remote          status
--------- ------------- ----------------- ----------- ------------ ------------
    55       cl-1122       active-full        Po55         Po55           up/up
le-2#show mlag
MLAG Configuration:
domain-id                          :          LEAVES-1-2
local-interface                    :            Vlan4094
peer-address                       :        172.16.101.1
peer-link                          :      Port-Channel78
hb-peer-address                    :         192.168.0.1
hb-peer-vrf                        :                MGMT
peer-config                        :          consistent

MLAG Status:
state                              :              Active
negotiation status                 :           Connected
peer-link status                   :                  Up
local-int status                   :                  Up
system-id                          :   52:01:00:27:03:91
dual-primary detection             :            Disabled
dual-primary interface errdisabled :               False

MLAG Ports:
Disabled                           :                   0
Configured                         :                   0
Inactive                           :                   0
Active-partial                     :                   0
Active-full                        :                   1

le-2#
```
