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
system jumbomtu 9216

key chain OSPF-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  description Router-ID
  ip address 10.0.1.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/1
  description to-LEAF-01_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.0/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown


interface Ethernet1/2
  description to-LEAF-02_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.2/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/3
  description to-LEAF-03_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.4/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.1.1
  passive-interface default
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail
end
copy running-config startup-config
```
---
## SPINE-02
```text
configure terminal
feature ospf
feature bfd
system jumbomtu 9216

key chain OSPF-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  description Router-ID
  ip address 10.0.2.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface Ethernet1/1
  description to-LEAF-01_Eth1/2
  no switchport
  mtu 9216
  ip address 10.2.2.0/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/2
  description to-LEAF-02_Eth1/2
  no switchport
  mtu 9216
  ip address 10.2.2.2/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/3
  description to-LEAF-03_Eth1/2
  no switchport
  mtu 9216
  ip address 10.2.2.4/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.2.1
  passive-interface default
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail
end
copy running-config startup-config
```
---
## LEAF-01
```text
configure terminal
feature ospf
feature bfd
system jumbomtu 9216

key chain OSPF-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  description Router-ID
  ip address 10.0.101.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description VTEP_and_Services
  ip address 10.1.101.1/32
  no shutdown

interface Ethernet1/1
  description to-SPINE-01_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.1/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/2
  description to-SPINE-02_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.2.1/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.101.1
  passive-interface default
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail

end
copy running-config startup-config
```
---
## LEAF-02
```text
configure terminal
feature ospf
feature bfd
system jumbomtu 9216

key chain OSPF-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  description Router-ID
  ip address 10.0.102.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description VTEP_and_Services
  ip address 10.1.102.1/32
  no shutdown

interface Ethernet1/1
  description to-SPINE-01_Eth1/2
  no switchport
  mtu 9216
  ip address 10.2.1.3/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/2
  description to-SPINE-02_Eth1/2
  no switchport
  mtu 9216
  ip address 10.2.2.3/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.102.1
  passive-interface default
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail

end
copy running-config startup-config
```
---
## LEAF-03
```text
configure terminal
feature ospf
feature bfd
system jumbomtu 9216

key chain OSPF-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  description Router-ID
  ip address 10.0.103.1/32
  ip router ospf UNDERLAY area 0.0.0.0
  no shutdown

interface loopback1
  description VTEP_and_Services
  ip address 10.1.103.1/32
  no shutdown

interface Ethernet1/1
  description to-SPINE-01_Eth1/3
  no switchport
  mtu 9216
  ip address 10.2.1.5/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

interface Ethernet1/2
  description to-SPINE-02_Eth1/3
  no switchport
  mtu 9216
  ip address 10.2.2.5/31
  ip ospf network point-to-point
  ip router ospf UNDERLAY area 0.0.0.0
  bfd interval 100 min_rx 100 multiplier 3
  ip ospf authentication key-chain OSPF-KEYS
  no bfd echo
  no shutdown

router ospf UNDERLAY
  router-id 10.0.103.1
  passive-interface default
  auto-cost reference-bandwidth 100000
  bfd
  log-adjacency-changes detail

end
copy running-config startup-config
```
---

