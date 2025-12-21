# Домашнее задание
Underlay. OSPF
## Цель:
Настроить OSPF для Underlay сети.

---

# Конфигурация OSPF

## SPINE-01
```text
configure terminal
feature ospf
feature bfd

! Глобальная настройка MTU
system jumbomtu 9216

interface loopback0
  description Router-ID
  ip address 10.0.1.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

! P2P к LEAF-01
interface Ethernet1/1
  description to-LEAF-01_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.0/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

! P2P к LEAF-02
interface Ethernet1/2
  description to-LEAF-02_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.2/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

! P2P к LEAF-03
interface Ethernet1/3
  description to-LEAF-03_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.4/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.1.1
  passive-interface default
  no passive-interface Ethernet1/1
  no passive-interface Ethernet1/2
  no passive-interface Ethernet1/3
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail
