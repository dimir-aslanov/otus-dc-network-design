# Домашнее задание
Underlay. IS-IS
## Цель:
Настроить IS-IS для Underlay сети.

---
# Схема
<img width="449" height="371" alt="image" src="https://github.com/user-attachments/assets/9125a61a-93fb-4a3c-a830-00f7c96b8eab" />

---
# Конфигурация IS=IS
## SPINE-01
```text
configure terminal
feature isis
feature bfd
system jumbomtu 9216

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.1.1/32
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/1
  no switchport
  mtu 9216
  ip address 10.2.1.0/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  no switchport
  mtu 9216
  ip address 10.2.1.2/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/3
  no switchport
  mtu 9216
  ip address 10.2.1.4/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0001.0000.0001.00
  is-type level-1
  log-adjacency-changes
  bfd
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1-2

end
copy running-config startup-config
```
## SPINE-02
```text
configure terminal
feature isis
feature bfd
system jumbomtu 9216

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.2.1/32
  ip router isis UNDERLAY
  isis network point-to-point
  no shutdown

interface Ethernet1/1
  no switchport
  mtu 9216
  ip address 10.2.2.0/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  no switchport
  mtu 9216
  ip address 10.2.2.2/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/3
  no switchport
  mtu 9216
  ip address 10.2.2.4/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0001.0000.0002.00
  is-type level-1
  log-adjacency-changes
  bfd
  metric-style wide
  address-family ipv4 unicast
  passive-interface loopback0
  authentication-type md5
  authentication key-chain ISIS-KEYS

end
copy running-config startup-config
```
## LEAF-01
```text
configure terminal
feature isis
feature bfd
system jumbomtu 9216

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.101.1/32
  ip router isis UNDERLAY
  isis network point-to-point
  no shutdown

interface loopback1
  ip address 10.1.101.1/32
  no shutdown

interface Ethernet1/1
  no switchport
  mtu 9216
  ip address 10.2.1.1/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  no switchport
  mtu 9216
  ip address 10.2.2.1/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0001.0000.0101.00
  is-type level-1
  log-adjacency-changes
  bfd
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1-2

end
copy running-config startup-config
```
## LEAF-02
```text
configure terminal
feature isis
feature bfd
system jumbomtu 9216

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.102.1/32
  ip router isis UNDERLAY
  isis network point-to-point
  no shutdown

interface loopback1
  ip address 10.1.102.1/32
  no shutdown

interface Ethernet1/1
  no switchport
  mtu 9216
  ip address 10.2.1.3/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  no switchport
  mtu 9216
  ip address 10.2.2.3/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0001.0000.0102.00
  is-type level-1
  log-adjacency-changes
  bfd
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1-2

end
copy running-config startup-config
```
## LEAF-03
```text
configure terminal
feature isis
feature bfd
system jumbomtu 9216

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.103.1/32
  ip router isis UNDERLAY
  isis network point-to-point
  no shutdown

interface loopback1
  ip address 10.1.103.1/32
  no shutdown

interface Ethernet1/1
  no switchport
  mtu 9216
  ip address 10.2.1.5/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  no switchport
  mtu 9216
  ip address 10.2.2.5/31
  ip router isis UNDERLAY
  isis circuit-type level-1
  isis network point-to-point
  no isis passive-interface level-1
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0001.0000.0103.00
  is-type level-1
  log-adjacency-changes
  bfd
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1-2

end
copy running-config startup-config
```
---
# Вывод CLI
## SPINE-01
```text
```
## SPINE-02
```text
```
## LEAF-01
```text
```
## LEAF-02
```text
```
## LEAF-03
```text
```
