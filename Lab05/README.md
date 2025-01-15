# Проектирование адресного пространства

## Цели работ

1. Настроить BGP peering между Leaf и Spine в AF l2vpn evpn
2. Настроить связанность между клиентами в первой зоне и убедитесь в её наличии
3. Зафиксировать в документации - план работы, адресное пространство, схему сети, конфигурацию устройств

## Выполнение Работ

### Топология

Реализованная схема:
![image](./MyScheme_OSPF.png)

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
router ospf 1 <br>
   router-id 10.2Y.255.1 <br>
   passive-interface default <br>
   no passive-interface Ethernet7 <br>
   no passive-interface Ethernet8 <br>
   redistribute connected route-map RM_OSPF_OUT <br>
   max-lsa 12000 <br>
<br>
interface Ethernet7 <br>
   description --- Spine-01 --- <br>
   ip ospf neighbor bfd <br>
   ip ospf network point-to-point <br>
   ip ospf area 0.0.0.10 <br>
<br>
interface Ethernet8 <br>
   description --- Spine-02 --- <br>
   ip ospf neighbor bfd <br>
   ip ospf network point-to-point <br>
   ip ospf area 0.0.0.10 <br>
<br>
route-map RM_OSPF_OUT permit 1 <br>
   match ip address prefix-list PL_OSPF_OUT <br>
<br>
ip prefix-list PL_OSPF_OUT seq 10 permit 10.2Y.255.1/32<br>
Где Y номер Leaf коммутатора в схеме <br>
</details>

### Настройка BGP в Overlay
