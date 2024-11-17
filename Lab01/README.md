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

Пространство выделялось из сети 10.0.0.0/8: \
P2P сети строились по схеме: \
10.XY.Z.0/30

Лупбеки по схеме: \
10.XY.255.1/32 Для маршрутизации \
10.10.255.1/32 Для точки рандеву \
10.20Y.1.1/32 Для VTEP лупбека на Leaf коммутаторах

Где: \
X - ряд коммутаторов. 1 - Spine, 2 - Leaf \
Y - Номер коммутатора в ряду \
Z - Номер Downlink/Uplink коммутатора

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
|Leaf-01|Ethernet1|Access Vlan 11|
|Leaf-01|Ethernet7|10.11.1.2/30|
|Leaf-01|Ethernet8|10.12.1.2/30|
|Leaf-01|Loopback1|10.21.255.1/32|
|Leaf-01|Loopback2|10.201.1.1/32|
|Leaf-01|Management1|10.255.1.11/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Leaf-02|Ethernet1|Access Vlan 11|
|Leaf-02|Ethernet7|10.11.2.2/30|
|Leaf-02|Ethernet8|10.12.2.2/30|
|Leaf-02|Loopback1|10.22.255.1/32|
|Leaf-02|Loopback2|10.202.1.1/32|
|Leaf-02|Management1|10.255.1.12/24|

|Device|Interface|IP Address/Mask|
|---|---|---|
|Leaf-03|Ethernet1|Access Vlan 11|
|Leaf-03|Ethernet7|10.11.3.2/30|
|Leaf-03|Ethernet8|10.12.3.2/30|
|Leaf-03|Loopback1|10.23.255.1/32|
|Leaf-03|Loopback2|10.203.1.1/32|
|Leaf-03|Management1|10.255.1.13/24|

### VLAN Таблица

|VLAN|Название|Связанные интерфейсы|Описание|
|---|---|---|---|
|255|MGMT|MGMT-01: Eth1-5| Для подключения к коммутаторам внутри фабрики с коммутатора MGMT-01. Сеть 10.255.1.0/24|
|11|Clients|Leaf-01,02,03:Eth1|Клиентское подключение|

### Конфигурация портов

<details>
<summary>Spine-01</summary>
<br>
interface Ethernet1 <br>
   description --- Leaf-01 --- <br>
   no switchport <br>
   ip address 10.11.1.1/30 <br>
<br>
interface Ethernet2 <br>
   description --- Leaf-02 --- <br>
   no switchport <br>
   ip address 10.11.2.1/30 <br>
<br>
interface Ethernet3 <br>
   description --- Leaf-03 --- <br>
   no switchport <br>
   ip address 10.11.3.1/30 <br>
<br>
interface Loopback0 <br>
   description --- For Routing --- <br>
   ip address 10.11.255.1/32 <br>
<br>
interface Loopback255 <br>
   description --- Rendezvous Point --- <br>
   ip address 10.10.255.1/32 <br>
<br>
interface Management1 <br>
   vrf MGMT <br>
   ip address 10.255.1.1/24 <br>
</details>

<details>
<summary>Spine-02</summary>
<br>
interface Ethernet1 <br>
   description --- Leaf-01 --- <br>
   no switchport <br>
   ip address 10.12.1.1/30 <br>
<br>
interface Ethernet2 <br>
   description --- Leaf-02 --- <br>
   no switchport <br>
   ip address 10.12.2.1/30 <br>
<br>
interface Ethernet3 <br>
   description --- Leaf-03 --- <br>
   no switchport <br>
   ip address 10.12.3.1/30 <br>
<br>
interface Loopback0 <br>
   description --- For Routing --- <br>
   ip address 10.12.255.1/32 <br>
<br>
interface Loopback255 <br>
   description --- Rendezvous Point --- <br>
   ip address 10.10.255.1/32 <br>
<br>
interface Management1 <br>
   vrf MGMT <br>
   ip address 10.255.1.2/24 <br>
</details>

<details>
<summary>Leaf-01</summary>
<br>
interface Ethernet1 <br>
   description --- Client --- <br>
   switchport access vlan 11 <br>
<br>
interface Ethernet7 <br>
   description --- Spine-01 --- <br>
   no switchport <br>
   ip address 10.11.1.2/30 <br>
<br>
interface Ethernet8 <br>
   description --- Spine-02 --- <br>
   no switchport <br>
   ip address 10.12.1.2/30 <br>
<br>
interface Loopback0 <br>
   description --- For Routing --- <br>
   ip address 10.21.255.1/32 <br>
<br>
interface Loopback1 <br>
   description --- VTEP Lo --- <br>
   ip address 10.201.1.1/32 <br>
<br>
interface Management1 <br>
   vrf MGMT <br>
   ip address 10.255.1.11/24 <br>
</details>

<details>
<summary>Leaf-02</summary>
<br>
interface Ethernet1 <br>
   description --- Client --- <br>
   switchport access vlan 11 <br>
<br>
interface Ethernet7 <br>
   description --- Spine-01 --- <br>
   no switchport <br>
   ip address 10.11.2.2/30 <br>
<br>
interface Ethernet8 <br>
   description --- Spine-02 --- <br>
   no switchport <br>
   ip address 10.12.2.2/30 <br>
<br>
interface Loopback0 <br>
   description --- For Routing --- <br>
   ip address 10.22.255.1/32 <br>
<br>
interface Loopback1 <br>
   description --- VTEP Lo --- <br>
   ip address 10.202.1.1/32 <br>
<br>
interface Management1 <br>
   vrf MGMT <br>
   ip address 10.255.1.12/24 <br>
</details>

<details>
<summary>Leaf-03</summary>
<br>
interface Ethernet1 <br>
   description --- Client --- <br>
   switchport access vlan 11 <br>
<br>
interface Ethernet7 <br>
   description --- Spine-01 --- <br>
   no switchport <br>
   ip address 10.11.3.2/30 <br>
<br>
interface Ethernet8 <br>
   description --- Spine-02 --- <br>
   no switchport <br>
   ip address 10.12.3.2/30 <br>
<br>
interface Loopback0 <br>
   description --- For Routing --- <br>
   ip address 10.23.255.1/32 <br>
<br>
interface Loopback1 <br>
   description --- VTEP Lo --- <br>
   ip address 10.203.1.1/32 <br>
<br>
interface Management1 <br>
   vrf MGMT <br>
   ip address 10.255.1.13/24 <br>
</details>

<details>
<summary>MGMT-01</summary>
<br>
interface Ethernet1 <br>
   switchport access vlan 255 <br>
<br>
interface Ethernet2 <br>
   switchport access vlan 255 <br>
<br>
interface Ethernet3 <br>
   switchport access vlan 255 <br>
<br>
interface Ethernet4 <br>
   switchport access vlan 255 <br>
<br>
interface Ethernet5 <br>
   switchport access vlan 255 <br>
<br>
interface Vlan255 <br>
   ip address 10.255.1.254/24 <br>
</details>

### Проверка IP связности

``` Spine-01
#### Leaf-01 #### \
Spine-01#ping 10.11.1.2 \
PING 10.11.1.2 (10.11.1.2) 72(100) bytes of data. \
80 bytes from 10.11.1.2: icmp_seq=1 ttl=64 time=4.17 ms \
80 bytes from 10.11.1.2: icmp_seq=2 ttl=64 time=3.61 ms \
80 bytes from 10.11.1.2: icmp_seq=3 ttl=64 time=3.77 ms \
80 bytes from 10.11.1.2: icmp_seq=4 ttl=64 time=11.0 ms \
--- 10.11.1.2 ping statistics --- \
5 packets transmitted, 4 received, 20% packet loss, time 41ms \
rtt min/avg/max/mdev = 3.618/5.650/11.032/3.114 ms, pipe 2, ipg/ewma 10.435/4.935 ms \

#### Leaf-02 #### \
Spine-01#ping 10.11.2.2 \
PING 10.11.2.2 (10.11.2.2) 72(100) bytes of data. \
80 bytes from 10.11.2.2: icmp_seq=1 ttl=64 time=4.77 ms \
80 bytes from 10.11.2.2: icmp_seq=2 ttl=64 time=8.12 ms \
80 bytes from 10.11.2.2: icmp_seq=3 ttl=64 time=5.20 ms \
80 bytes from 10.11.2.2: icmp_seq=4 ttl=64 time=5.36 ms \
80 bytes from 10.11.2.2: icmp_seq=5 ttl=64 time=8.73 ms \
--- 10.11.2.2 ping statistics --- \
5 packets transmitted, 5 received, 0% packet loss, time 34ms \
rtt min/avg/max/mdev = 4.773/6.438/8.736/1.649 ms, ipg/ewma 8.538/5.654 ms

#### Leaf-03 #### \
Spine-01#ping 10.11.3.2 Leaf-03 \
PING 10.11.3.2 (10.11.3.2) 72(100) bytes of data. \
80 bytes from 10.11.3.2: icmp_seq=1 ttl=64 time=4.47 ms \
80 bytes from 10.11.3.2: icmp_seq=2 ttl=64 time=6.65 ms \
80 bytes from 10.11.3.2: icmp_seq=3 ttl=64 time=3.72 ms \
80 bytes from 10.11.3.2: icmp_seq=4 ttl=64 time=3.62 ms \
80 bytes from 10.11.3.2: icmp_seq=5 ttl=64 time=5.47 ms \
--- 10.11.3.2 ping statistics --- \
5 packets transmitted, 5 received, 0% packet loss, time 30ms \
rtt min/avg/max/mdev = 3.622/4.789/6.651/1.146 ms, ipg/ewma 7.502/4.619 ms \
```

``` Spine-02
asd
```
