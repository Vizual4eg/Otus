# Проектирование адресного пространства

## Цели работ

1. Настроить каждого клиента в своем VNI
2. Настроить маршрутизацию между клиентами.
3. Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

## Выполнение Работ

### Топология

Реализованная схема OSPF:

![image](./MyScheme_OSPF.png)

Реализованная схема BGP EVPN:

![image](./MyScheme_BGP_EVPN.png)

### [Адресное Пространство](/Lab01/README.md#%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%BD%D0%BE%D0%B5-%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D1%81%D1%82%D0%B2%D0%BE)

### Настройки OSPF в Underlay

<details>
<summary>Spine's</summary>
<br>
router ospf 1 <br>
   router-id 10.1X.255.1 <br>
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
<details>
<summary>Leaf's</summary>
<br>
router bgp 65099 <br>
   maximum-paths 4 ecmp 64 <br>
   neighbor SPINE_GROUP peer group <br>
   neighbor SPINE_GROUP remote-as 65099 <br>
   neighbor SPINE_GROUP bfd <br>
   neighbor SPINE_GROUP route-map RM_BGP_IN in <br>
   neighbor SPINE_GROUP route-map RM_BGP_OUT out <br>
   neighbor 10.1X.1.1 peer group SPINE_GROUP <br>
   neighbor 10.1X.1.1 peer group SPINE_GROUP <br>
   network 10.2Y.255.1/32 <br>
<br>
Где X номер Spine коммутатора в схеме <br>
Где Y номер Leaf коммутатора в схеме <br>
</details>

### Настройка BGP в Overlay

<details>
    <summary>Spine's</summary>
    <br>
    router bgp 65000<br>
     maximum-paths 2 ecmp 64<br>
   neighbor EVPN peer group<br>
   neighbor EVPN update-source Loopback0<br>
   neighbor EVPN ebgp-multihop 3<br>
   neighbor EVPN send-community extended<br>
   neighbor EVPN maximum-routes 12000 warning-only<br>
   neighbor 10.21.255.1 peer group EVPN<br>
   neighbor 10.21.255.1 remote-as 65001<br>
   neighbor 10.22.255.1 peer group EVPN<br>
   neighbor 10.22.255.1 remote-as 65002<br>
   neighbor 10.23.255.1 peer group EVPN<br>
   neighbor 10.23.255.1 remote-as 65003<br>
   !<br>
   address-family evpn<br>
      neighbor EVPN activate<br>
</details>
<details>
    <summary>Leaf's</summary>
    router bgp 6500Y<br>
    maximum-paths 2 ecmp 64<br>
   neighbor EVPN peer group<br>
   neighbor EVPN remote-as 65000<br>
   neighbor EVPN update-source Loopback0<br>
   neighbor EVPN ebgp-multihop 3<br>
   neighbor EVPN send-community extended<br>
   neighbor EVPN maximum-routes 12000 warning-only<br>
   neighbor 10.11.255.1 peer group EVPN<br>
   neighbor 10.12.255.1 peer group EVPN<br>
   !<br>
   vlan 1001<br>
      rd auto<br>
      route-target both 1001:10001<br>
      redistribute learned<br>
   !<br>
   vlan 1002<br>
      rd auto<br>
      route-target both 1002:10002<br>
      redistribute learned<br>
   !<br>
   address-family evpn<br>
      neighbor EVPN activate<br>
   !<br>
   address-family ipv4<br>
      network 10.20Y.1.1/32<br><br>
    Где Y номер Leaf в схеме<br>
</details>

### Настройка VXLAN интерфейса на Leaf

interface Vxlan1

   vxlan source-interface Loopback1

   vxlan udp-port 4789

   vxlan vlan 1001 vni 10001

   vxlan vlan 1002 vni 10002

   vxlan learn-restrict any

### Таблица VLAN и VNI

|Vlan Name|Vlan ID|VNI|
|---|---|---|
|Customer-01|1001|10001|
|Customer-02|1002|10002|

### Настройки VLAN интерфейсов на Leaf

``` Leaf's
interface Vlan1001
   description --- Customer-01 ---
   ip address virtual 192.168.1.254/24
   
interface Vlan1002
   description --- Customer-02 ---
   ip address virtual 192.168.2.254/24
```

### Настройки интерфейсов на Leaf

| Leaf    | Interface | Vlan | Номер PC |
| ------- | --------- | ---- | -------- |
| Leaf-01 | Ethernet1 | 1001 | PC-01    |
| Leaf-02 | Ethernet1 | 1002 | PC-02    |
| Leaf-03 | Ethernet1 | 1001 | PC-03    |
| Leaf-03 | Ethernet2 | 1002 | PC-04    |

### Настройка AnyCastGateway

```Virtual-mac
ip virtual-router mac-address 00:00:00:01:00:00
```

### BGP EVPN связность

#### Leaf-01

``` Leaf-01
Leaf-01#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.201.1.1, local AS number 65001
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.11.255.1 4 65000           5087      5083    0    0    2d23h Estab   10     10
  10.12.255.1 4 65000           5058      5073    0    0    2d23h Estab   10     10
```

#### Leaf-02

``` Leaf-02
Leaf-02#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.202.1.1, local AS number 65002
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.11.255.1 4 65000           5076      5075    0    0    2d23h Estab   10     10
  10.12.255.1 4 65000           5069      5065    0    0    2d23h Estab   10     10
```

#### Leaf-03

``` Leaf-03
Leaf-03#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.203.1.1, local AS number 65003
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.11.255.1 4 65000           5066      5060    0    0    2d23h Estab   4      4
  10.12.255.1 4 65000           5065      5070    0    0    2d23h Estab   4      4
```

#### Spine-01

``` Spine-01
Spine-01#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.11.255.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.21.255.1 4 65001           5087      5089    0    0    2d23h Estab   2      2
  10.22.255.1 4 65002           5078      5079    0    0    2d23h Estab   2      2
  10.23.255.1 4 65003           5060      5066    0    0    2d23h Estab   2      2
```

#### Spine-02

``` Spine-02
Spine-02#sh bgp evpn summary
BGP summary information for VRF default
Router identifier 10.12.255.1, local AS number 65000
Neighbor Status Codes: m - Under maintenance
  Neighbor    V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.21.255.1 4 65001           5079      5062    0    0    2d23h Estab   2      2
  10.22.255.1 4 65002           5070      5072    0    0    2d23h Estab   2      2
  10.23.255.1 4 65003           5070      5066    0    0    2d23h Estab   2      2
```

### Проверка связности между ПК

#### PC-01 (1001 VLAN) -> PC02 (1002 VLAN)

``` PC-01
NAME   IP/MASK              GATEWAY
PC-01  192.168.1.1/24       192.168.1.254

PC-01> show arp
00:00:00:01:00:00  192.168.1.254 expires in 119 seconds

PC-01> ping 192.168.2.2
84 bytes from 192.168.2.2 icmp_seq=1 ttl=63 time=295.031 ms
84 bytes from 192.168.2.2 icmp_seq=2 ttl=63 time=20.336 ms
84 bytes from 192.168.2.2 icmp_seq=3 ttl=63 time=20.202 ms
84 bytes from 192.168.2.2 icmp_seq=4 ttl=63 time=20.471 ms


NAME   IP/MASK              GATEWAY
PC-02  192.168.2.2/24       192.168.2.254

PC-02> show arp
00:00:00:01:00:00  192.168.2.254 expires in 118 seconds

PC-02> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=63 time=23.451 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=63 time=24.650 ms

```

#### Leaf-02 BGP EVPN Таблица

``` Leaf-02

Leaf-02#sh bgp evpn
BGP routing table information for VRF default
Router identifier 10.202.1.1, local AS number 65002
Route status codes: * - valid, > - active, S - Stale, E - ECMP head, e - ECMP
                    c - Contributing to ECMP, % - Pending BGP convergence
Origin codes: i - IGP, e - EGP, ? - incomplete
AS Path Attributes: Or-ID - Originator ID, C-LST - Cluster List, LL Nexthop - Link Local Nexthop

          Network                Next Hop              Metric  LocPref Weight  Path
 * >Ec    RD: 10.201.1.1:1001 mac-ip 0050.7966.688a
                                 10.201.1.1            -       100     0       65000 65001 i
 *  ec    RD: 10.201.1.1:1001 mac-ip 0050.7966.688a
                                 10.201.1.1            -       100     0       65000 65001 i
 * >Ec    RD: 10.201.1.1:1001 mac-ip 0050.7966.688a 192.168.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 *  ec    RD: 10.201.1.1:1001 mac-ip 0050.7966.688a 192.168.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 * >      RD: 10.202.1.1:1002 mac-ip 0050.7966.688b
                                 -                     -       -       0       i
 * >      RD: 10.202.1.1:1002 mac-ip 0050.7966.688b 192.168.2.2
                                 -                     -       -       0       i
 * >Ec    RD: 10.201.1.1:1001 imet 10.201.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 *  ec    RD: 10.201.1.1:1001 imet 10.201.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 * >Ec    RD: 10.201.1.1:1002 imet 10.201.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 *  ec    RD: 10.201.1.1:1002 imet 10.201.1.1
                                 10.201.1.1            -       100     0       65000 65001 i
 * >      RD: 10.202.1.1:1001 imet 10.202.1.1
                                 -                     -       -       0       i
 * >      RD: 10.202.1.1:1002 imet 10.202.1.1
                                 -                     -       -       0       i
 * >Ec    RD: 65003:10001 imet 10.203.1.1
                                 10.203.1.1            -       100     0       65000 65003 i
 *  ec    RD: 65003:10001 imet 10.203.1.1
                                 10.203.1.1            -       100     0       65000 65003 i
 * >Ec    RD: 10.203.1.1:1002 imet 10.203.1.1
                                 10.203.1.1            -       100     0       65000 65003 i
 *  ec    RD: 10.203.1.1:1002 imet 10.203.1.1
                                 10.203.1.1            -       100     0       65000 65003 i
```

#### PC-03->04

``` PC-03
NAME   IP/MASK              GATEWAY
PC-03  192.168.1.3/24       192.168.1.254

PC-03> sh arp
00:00:00:01:00:00  192.168.1.254 expires in 66 seconds

PC-03> ping 192.168.2.4
84 bytes from 192.168.2.4 icmp_seq=1 ttl=63 time=25.662 ms
84 bytes from 192.168.2.4 icmp_seq=2 ttl=63 time=8.204 ms
84 bytes from 192.168.2.4 icmp_seq=3 ttl=63 time=8.474 ms


NAME   IP/MASK              GATEWAY
PC-04  192.168.2.4/24       192.168.2.254

PC-04> sh arp
00:00:00:01:00:00  192.168.2.254 expires in 66 seconds

PC-04> ping 192.168.1.3
84 bytes from 192.168.1.3 icmp_seq=1 ttl=63 time=27.079 ms
84 bytes from 192.168.1.3 icmp_seq=2 ttl=63 time=6.490 ms
84 bytes from 192.168.1.3 icmp_seq=3 ttl=63 time=6.848 ms
84 bytes from 192.168.1.3 icmp_seq=4 ttl=63 time=14.550 ms
```
