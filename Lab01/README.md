# Проектирование адресного пространства

## Цели работ

1. Собрать топологию CLOS, как на схеме:\
![image](./DCScheme.avif)
2. Распределить адресное пространство для Underlay сети
3. Зафиксировать план работ и адресное пространство

## Выполнение Работ

### Топология CLOS

Реализованная схема:\
![image](./MyScheme.png)

### Адресное пространство

|Device|Interface|Vlan/IP Address/Mask|
|---|---|---|
|MGMT-01|Ethernet1|Access Vlan 255|
|MGMT-01|Ethernet2|Access Vlan 255|
|MGMT-01|Ethernet3|Access Vlan 255|
|MGMT-01|Ethernet4|Access Vlan 255|
|MGMT-01|Ethernet5|Access Vlan 255|
|MGMT-01|Vlan 255|10.255.1.254/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Spine-01|Ethernet1|10.11.1.1/30|
|Spine-01|Ethernet2|10.11.2.1/30|
|Spine-01|Ethernet3|10.11.3.1/30|
|Spine-01|Loopback1|10.11.255.1/32|
|Spine-01|Loopback255|10.10.255.1/32|
|Spine-01|Management1|10.255.1.1/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Spine-02|Ethernet1|10.12.1.1/30|
|Spine-02|Ethernet2|10.12.2.1/30|
|Spine-02|Ethernet3|10.12.3.1/30|
|Spine-02|Loopback1|10.12.255.1/32|
|Spine-02|Loopback255|10.10.255.1/32|
|Spine-02|Management1|10.255.1.2/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Leaf-01|Ethernet7|10.11.1.2/30|
|Leaf-01|Ethernet8|10.12.1.2/30|
|Leaf-01|Loopback1|10.21.255.1/32|
|Leaf-01|Loopback2|10.101.1.1/32|
|Leaf-01|Management1|10.255.1.11/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Leaf-01|Ethernet7|10.11.2.2/30|
|Leaf-01|Ethernet8|10.12.2.2/30|
|Leaf-01|Loopback1|10.22.255.1/32|
|Leaf-01|Loopback2|10.102.1.1/32|
|Leaf-01|Management1|10.255.1.12/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Leaf-01|Ethernet7|10.11.3.2/30|
|Leaf-01|Ethernet8|10.12.3.2/30|
|Leaf-01|Loopback1|10.23.255.1/32|
|Leaf-01|Loopback2|10.103.1.1/32|
|Leaf-01|Management1|10.255.1.13/24|

### VLAN Таблица

|VLAN|Название|Связанные интерфейсы|Описание|
|---|---|---|---|
|255|MGMT|MGMT-01: Eth-1-5| Для подключения к коммутаторам внутри фабрики с коммутатора MGMT-01|
|11|Clients|Leaf-01:Eth1, Leaf-02:Eth1, Leaf-03:Eth1|Клиентское подключение|
