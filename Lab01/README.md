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
|xxxxxxxxxx Spine-02#sh isis neighborsInstance  VRF      System Id        Type Interface          SNPA              State Hold timeKITEZH    default  Leaf-01          L1   Ethernet1          50:0:ac:b4:80:59  UP    28KITEZH    default  Leaf-01          L2   Ethernet1          50:0:ac:b4:80:59  UP    29KITEZH    default  Leaf-02          L1   Ethernet2          50:59:58:47:be:14 UP    9KITEZH    default  Leaf-02          L2   Ethernet2          50:59:58:47:be:14 UP    9KITEZH    default  Leaf-03          L1   Ethernet3          50:4d:7d:ff:ee:e5 UP    7KITEZH    default  Leaf-03          L2   Ethernet3          50:4d:7d:ff:ee:e5 UP    7​​Spine-02#sh bfd peersVRF name: default-----------------DstAddr        MyDisc    YourDisc  Interface/Transport    Type          LastUp---------- ----------- ----------- -------------------- ------- ---------------10.12.1.2  2017224285  3833958468        Ethernet1(13)  normal  11/30/24 18:5310.12.2.2  2269395687   602051689        Ethernet2(14)  normal  11/30/24 18:5310.12.3.2  2002387184  3882910188        Ethernet3(15)  normal  11/30/24 18:53​   LastDown            LastDiag    State-------------- ------------------- -----         NA       No Diagnostic       Up         NA       No Diagnostic       Up         NA       No Diagnostic       Up​​Spine-02#sh ip route isis I L1     10.11.1.0/30 [115/20] via 10.12.1.2, Ethernet1 I L1     10.11.2.0/30 [115/20] via 10.12.2.2, Ethernet2 I L1     10.11.3.0/30 [115/20] via 10.12.3.2, Ethernet3 I L1     10.11.255.1/32 [115/30] via 10.12.1.2, Ethernet1                                  via 10.12.2.2, Ethernet2                                  via 10.12.3.2, Ethernet3 I L1     10.21.255.1/32 [115/20] via 10.12.1.2, Ethernet1 I L1     10.22.255.1/32 [115/20] via 10.12.2.2, Ethernet2 I L1     10.23.255.1/32 [115/20] via 10.12.3.2, Ethernet3​​### LOOPBACK PING ###### Leaf-01Spine-02#ping 10.21.255.1  repeat 1 80 bytes from 10.21.255.1: icmp_seq=1 ttl=64 time=5.24 ms​### Leaf-02Spine-02#ping 10.22.255.1  repeat 1 80 bytes from 10.22.255.1: icmp_seq=1 ttl=64 time=4.96 ms ### Leaf-03Spine-02#ping 10.23.255.1  repeat 1 80 bytes from 10.23.255.1: icmp_seq=1 ttl=64 time=6.06 ms​### Spine-01Spine-02#ping 10.11.255.1  repeat 1 80 bytes from 10.11.255.1: icmp_seq=1 ttl=63 time=8.75 msSpine-02|Ethernet1|Access Vlan 255|
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

### Spine-01

``` Spine-01
#### Leaf-01 ####
Spine-01#ping 10.11.1.2
PING 10.11.1.2 (10.11.1.2) 72(100) bytes of data.
80 bytes from 10.11.1.2: icmp_seq=1 ttl=64 time=4.17 ms
80 bytes from 10.11.1.2: icmp_seq=2 ttl=64 time=3.61 ms
80 bytes from 10.11.1.2: icmp_seq=3 ttl=64 time=3.77 ms
80 bytes from 10.11.1.2: icmp_seq=4 ttl=64 time=11.0 ms
--- 10.11.1.2 ping statistics ---
5 packets transmitted, 4 received, 20% packet loss, time 41ms
rtt min/avg/max/mdev = 3.618/5.650/11.032/3.114 ms, pipe 2, ipg/ewma 10.435/4.935 ms

#### Leaf-02 ####
Spine-01#ping 10.11.2.2 
PING 10.11.2.2 (10.11.2.2) 72(100) bytes of data.
80 bytes from 10.11.2.2: icmp_seq=1 ttl=64 time=4.77 ms
80 bytes from 10.11.2.2: icmp_seq=2 ttl=64 time=8.12 ms
80 bytes from 10.11.2.2: icmp_seq=3 ttl=64 time=5.20 ms
80 bytes from 10.11.2.2: icmp_seq=4 ttl=64 time=5.36 ms
80 bytes from 10.11.2.2: icmp_seq=5 ttl=64 time=8.73 ms
--- 10.11.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 34ms
rtt min/avg/max/mdev = 4.773/6.438/8.736/1.649 ms, ipg/ewma 8.538/5.654 ms

#### Leaf-03 ####
Spine-01#ping 10.11.3.2 Leaf-03
PING 10.11.3.2 (10.11.3.2) 72(100) bytes of data.
80 bytes from 10.11.3.2: icmp_seq=1 ttl=64 time=4.47 ms
80 bytes from 10.11.3.2: icmp_seq=2 ttl=64 time=6.65 ms
80 bytes from 10.11.3.2: icmp_seq=3 ttl=64 time=3.72 ms
80 bytes from 10.11.3.2: icmp_seq=4 ttl=64 time=3.62 ms
80 bytes from 10.11.3.2: icmp_seq=5 ttl=64 time=5.47 ms
--- 10.11.3.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 30ms
rtt min/avg/max/mdev = 3.622/4.789/6.651/1.146 ms, ipg/ewma 7.502/4.619 ms
```

### Spine-02

``` Spine-02
#### Leaf-01 ####
Spine-02#ping 10.12.1.2
PING 10.12.1.2 (10.12.1.2) 72(100) bytes of data.
80 bytes from 10.12.1.2: icmp_seq=1 ttl=64 time=48.0 ms
80 bytes from 10.12.1.2: icmp_seq=2 ttl=64 time=44.9 ms
80 bytes from 10.12.1.2: icmp_seq=3 ttl=64 time=42.0 ms
80 bytes from 10.12.1.2: icmp_seq=4 ttl=64 time=38.1 ms
80 bytes from 10.12.1.2: icmp_seq=5 ttl=64 time=34.7 ms
--- 10.12.1.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 34.745/41.580/48.039/4.728 ms, pipe 5, ipg/ewma 11.416/44.460 ms

#### Leaf-02 ####
Spine-02#ping 10.12.2.2
PING 10.12.2.2 (10.12.2.2) 72(100) bytes of data.
80 bytes from 10.12.2.2: icmp_seq=1 ttl=64 time=30.4 ms
80 bytes from 10.12.2.2: icmp_seq=2 ttl=64 time=27.0 ms
80 bytes from 10.12.2.2: icmp_seq=3 ttl=64 time=18.8 ms
80 bytes from 10.12.2.2: icmp_seq=4 ttl=64 time=15.3 ms
80 bytes from 10.12.2.2: icmp_seq=5 ttl=64 time=5.36 ms
--- 10.12.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 137ms
rtt min/avg/max/mdev = 5.369/19.419/30.495/8.885 ms, pipe 2, ipg/ewma 34.430/24.294 ms

#### Leaf-03 ####
Spine-02#ping 10.12.3.2
PING 10.12.3.2 (10.12.3.2) 72(100) bytes of data.
80 bytes from 10.12.3.2: icmp_seq=1 ttl=64 time=98.9 ms
80 bytes from 10.12.3.2: icmp_seq=2 ttl=64 time=66.8 ms
80 bytes from 10.12.3.2: icmp_seq=3 ttl=64 time=76.6 ms
80 bytes from 10.12.3.2: icmp_seq=4 ttl=64 time=63.0 ms
80 bytes from 10.12.3.2: icmp_seq=5 ttl=64 time=5.42 ms
--- 10.12.3.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 183ms
rtt min/avg/max/mdev = 5.426/62.182/98.956/31.003 ms, pipe 4, ipg/ewma 45.819/78.512 ms
```
