# Проектирование адресного пространства

## Цели работ

1. Настроить OSPF в Underlay сети, для IP связанности между всеми сетевыми устройствами.
2. Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств
3. Убедиться в наличии IP связанности между устройствами в OSFP домене

## Выполнение Работ

### Топология

Реализованная схема:\
![image](./MyScheme_OSPF.png)

### Настройки OSPF

<details>
<summary>Spine's</summary>
<br>
router ospf 1 <br>
   passive-interface default <br>
   no passive-interface Ethernet1 <br>
   no passive-interface Ethernet2 <br>
   no passive-interface Ethernet3 <br>
   redistribute connected route-map RM_OSPF_OUT <br>
   max-lsa 12000 <br>
<br>
interface Ethernet1 <br>
   description --- Leaf-01 --- <br>
   ip ospf neighbor bfd <br>
   ip ospf network point-to-point <br>
   ip ospf area 0.0.0.10 <br>
<br>
interface Ethernet2 <br>
   description --- Leaf-02 --- <br>
   ip ospf neighbor bfd <br>
   ip ospf network point-to-point <br>
   ip ospf area 0.0.0.10 <br>
<br>
interface Ethernet3 <br>
   description --- Leaf-03 --- <br>
   ip ospf neighbor bfd <br>
   ip ospf network point-to-point <br>
   ip ospf area 0.0.0.10 <br>
<br>
route-map RM_OSPF_OUT permit 1 <br>
   match ip address prefix-list PL_OSPF_OUT <br>
<br>
ip prefix-list PL_OSPF_OUT seq 10 permit 10.1X.255.1/32<br>
Где X номер Spine коммутатора в схеме <br>
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
