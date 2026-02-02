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
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.0/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.2/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/3
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.4/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0011.0000.0000.0001.00
  is-type level-1
  log-adjacency-changes
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1

end
copy running-config startup-config
```
## SPINE-02
```text
configure terminal
feature isis
feature bfd

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.2.1/32
  ip router isis UNDERLAY
  no shutdown

interface Ethernet1/1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.0/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.2/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/3
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.4/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable - # не работает в лабе ISIS с включенным BFD  
  no bfd echo
  no shutdown

router isis UNDERLAY
  net 49.0011.0000.0000.0002.00
  is-type level-1
  log-adjacency-changes
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1

end
copy running-config startup-config
```
## LEAF-01
```text
configure terminal
feature isis
feature bfd

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.101.1/32
  ip router isis UNDERLAY
  no shutdown

interface loopback1
  ip address 10.1.101.1/32
  no shutdown

nterface Ethernet1/1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.1/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.1/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

router isis UNDERLAY
  net 49.0011.0000.0000.0101.00
  is-type level-1
  log-adjacency-changes
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1

end
copy running-config startup-config
```
## LEAF-02
```text
configure terminal
feature isis
feature bfd

key chain ISIS-KEYS
  key 1
    key-string Network123
    cryptographic-algorithm md5

interface loopback0
  ip address 10.0.102.1/32
  ip router isis UNDERLAY
  no shutdown

interface loopback1
  ip address 10.1.102.1/32
  no shutdown

nterface Ethernet1/1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.3/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.3/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

router isis UNDERLAY
  net 49.0011.0000.0000.0102.00
  is-type level-1
  log-adjacency-changes
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1

end
copy running-config startup-config
```
## LEAF-03
```text
configure terminal
feature isis
feature bfd

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

nterface Ethernet1/1
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.5/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

interface Ethernet1/2
  no switchport
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.5/31
  isis network point-to-point
  isis circuit-type level-1
  isis authentication-type md5
  isis authentication key-chain ISIS-KEYS
  ip router isis UNDERLAY
  no isis passive-interface level-1
  isis bfd disable # не работает в лабе ISIS с включенным BFD  
  no shutdown

router isis UNDERLAY
  net 49.0011.0000.0000.0103.00
  is-type level-1
  log-adjacency-changes
  authentication-type md5 level-1
  authentication key-chain ISIS-KEYS level-1
  address-family ipv4 unicast
    bfd
  passive-interface default level-1

end
copy running-config startup-config
```
---
# Что не получилось в лабе
### BFD ISIS - при включении сразу падает ISIS 

# Вывод CLI

```
## LEAF-01
```text
LEAF-01(config-if)# sh isis adjacency
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE-01        N/A             1      UP     00:00:27   Ethernet1/1
SPINE-02        N/A             1      UP     00:00:29   Ethernet1/2

LEAF-01(config-if)# sh isis database l1
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  SPINE-01.00-00        0x00000043   0xCEDE    1065       0/0/0/1
  SPINE-02.00-00        0x00000032   0x4B13    1080       0/0/0/1
  LEAF-01.00-00       * 0x0000002A   0x58C8    743        0/0/0/1
  LEAF-02.00-00         0x0000002D   0x393E    1005       0/0/0/1
  LEAF-03.00-00         0x0000002B   0x8B1D    1086       0/0/0/1

LEAF-01(config-if)# sh ip route isis-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/41], 00:22:07, isis-UNDERLAY, L1
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/41], 00:07:49, isis-UNDERLAY, L1
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [115/81], 00:03:49, isis-UNDERLAY, L1
    *via 10.2.2.0, Eth1/2, [115/81], 00:03:25, isis-UNDERLAY, L1
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [115/81], 00:02:26, isis-UNDERLAY, L1
    *via 10.2.2.0, Eth1/2, [115/81], 00:01:59, isis-UNDERLAY, L1
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/80], 00:05:59, isis-UNDERLAY, L1
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [115/80], 00:05:40, isis-UNDERLAY, L1
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/80], 00:07:39, isis-UNDERLAY, L1
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [115/80], 00:07:49, isis-UNDERLAY, L1
```
## LEAF-02
```text
LEAF-02(config-if)# sh isis adjacency
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE-01        N/A             1      UP     00:00:30   Ethernet1/1
SPINE-02        N/A             1      UP     00:00:24   Ethernet1/2

LEAF-02(config-if)# sh isis database l1
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  SPINE-01.00-00        0x00000043   0xCEDE    924        0/0/0/1
  SPINE-02.00-00        0x00000032   0x4B13    939        0/0/0/1
  LEAF-01.00-00         0x0000002B   0x722E    1171       0/0/0/1
  LEAF-02.00-00       * 0x0000002D   0x393E    866        0/0/0/1
  LEAF-03.00-00         0x0000002B   0x8B1D    944        0/0/0/1

LEAF-02(config-if)# sh ip route isis-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [115/41], 00:06:31, isis-UNDERLAY, L1
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [115/41], 00:06:08, isis-UNDERLAY, L1
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [115/81], 00:06:31, isis-UNDERLAY, L1
    *via 10.2.2.2, Eth1/2, [115/81], 00:06:08, isis-UNDERLAY, L1
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [115/81], 00:05:08, isis-UNDERLAY, L1
    *via 10.2.2.2, Eth1/2, [115/81], 00:04:40, isis-UNDERLAY, L1
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [115/80], 00:06:31, isis-UNDERLAY, L1
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [115/80], 00:06:31, isis-UNDERLAY, L1
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [115/80], 00:06:08, isis-UNDERLAY, L1
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [115/80], 00:06:08, isis-UNDERLAY, L1

```
## LEAF-03
```text
EAF-03(config-if)# sh isis adjacency
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
SPINE-01        N/A             1      UP     00:00:25   Ethernet1/1
SPINE-02        N/A             1      UP     00:00:30   Ethernet1/2

LEAF-03(config-if)# sh isis database L1
IS-IS Process: UNDERLAY LSP database VRF: default
IS-IS Level-1 Link State Database
  LSPID                 Seq Number   Checksum  Lifetime   A/P/O/T
  SPINE-01.00-00        0x00000043   0xCEDE    1132       0/0/0/1
  SPINE-02.00-00        0x00000032   0x4B13    1147       0/0/0/1
  LEAF-01.00-00         0x0000002A   0x58C8    808        0/0/0/1
  LEAF-02.00-00         0x0000002D   0x393E    1072       0/0/0/1
  LEAF-03.00-00       * 0x0000002B   0x8B1D    1154       0/0/0/1

LEAF-03(config-if)# sh ip route isis-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [115/41], 00:01:24, isis-UNDERLAY, L1
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [115/41], 00:01:08, isis-UNDERLAY, L1
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [115/81], 00:01:24, isis-UNDERLAY, L1
    *via 10.2.2.4, Eth1/2, [115/81], 00:01:08, isis-UNDERLAY, L1
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [115/81], 00:01:24, isis-UNDERLAY, L1
    *via 10.2.2.4, Eth1/2, [115/81], 00:01:08, isis-UNDERLAY, L1
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [115/80], 00:01:24, isis-UNDERLAY, L1
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [115/80], 00:01:24, isis-UNDERLAY, L1
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [115/80], 00:01:08, isis-UNDERLAY, L1
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [115/80], 00:01:08, isis-UNDERLAY, L1

```
