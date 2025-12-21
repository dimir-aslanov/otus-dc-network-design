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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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
  no ip ospf passive-interface
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

# Вывод CLI

## SPINE-01
```text
SPINE-01# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.101.1        1 FULL/ -          00:02:09 10.2.1.1        Eth1/1
 10.0.102.1        1 FULL/ -          00:01:19 10.2.1.3        Eth1/2
 10.0.103.1        1 FULL/ -          00:01:12 10.2.1.5        Eth1/3

SPINE-01# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.2.1/32, ubest/mbest: 3/0
    *via 10.2.1.1, Eth1/1, [110/201], 00:02:47, ospf-UNDERLAY, intra
    *via 10.2.1.3, Eth1/2, [110/201], 00:02:44, ospf-UNDERLAY, intra
    *via 10.2.1.5, Eth1/3, [110/201], 00:02:38, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 1/0
    *via 10.2.1.1, Eth1/1, [110/101], 00:04:24, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 1/0
    *via 10.2.1.3, Eth1/2, [110/101], 00:03:34, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 1/0
    *via 10.2.1.5, Eth1/3, [110/101], 00:03:27, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.1.1, Eth1/1, [110/200], 00:04:12, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.1.3, Eth1/2, [110/200], 00:03:34, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.1.5, Eth1/3, [110/200], 00:03:27, ospf-UNDERLAY, intra

```

## SPINE-02
```text
SPINE-02# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 3
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.101.1        1 FULL/ -          00:01:59 10.2.2.1        Eth1/1
 10.0.102.1        1 FULL/ -          00:01:54 10.2.2.3        Eth1/2
 10.0.103.1        1 FULL/ -          00:01:49 10.2.2.5        Eth1/3

SPINE-02#  sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 3/0
    *via 10.2.2.1, Eth1/1, [110/201], 00:03:53, ospf-UNDERLAY, intra
    *via 10.2.2.3, Eth1/2, [110/201], 00:03:53, ospf-UNDERLAY, intra
    *via 10.2.2.5, Eth1/3, [110/201], 00:03:48, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 1/0
    *via 10.2.2.1, Eth1/1, [110/101], 00:03:53, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 1/0
    *via 10.2.2.3, Eth1/2, [110/101], 00:03:53, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 1/0
    *via 10.2.2.5, Eth1/3, [110/101], 00:03:48, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.2.1, Eth1/1, [110/200], 00:03:53, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.2.3, Eth1/2, [110/200], 00:03:53, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.2.5, Eth1/3, [110/200], 00:03:48, ospf-UNDERLAY, intra
```

## LEAF-01
```text
LEAF-01# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          00:06:22 10.2.1.0        Eth1/1
 10.0.2.1          1 FULL/ -          00:04:41 10.2.2.0        Eth1/2

LEAF-01# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/101], 00:06:55, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/101], 00:05:18, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 00:06:05, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 00:05:15, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 00:05:58, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 00:05:10, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/200], 00:06:17, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/200], 00:06:11, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/200], 00:05:18, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/200], 00:05:16, ospf-UNDERLAY, intra
```

## LEAF-02
```text
LEAF-02# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          00:07:25 10.2.1.2        Eth1/1
 10.0.2.1          1 FULL/ -          00:06:29 10.2.2.2        Eth1/2

LEAF-02# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/101], 00:07:33, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/101], 00:06:42, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 00:07:33, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 00:06:42, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 00:07:25, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 00:06:37, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/200], 00:07:33, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/200], 00:07:33, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/200], 00:06:42, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/200], 00:06:42, ospf-UNDERLAY, intra
```
## LEAF-03
```text
LEAF-03# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          00:07:55 10.2.1.4        Eth1/1
 10.0.2.1          1 FULL/ -          00:07:01 10.2.2.4        Eth1/2

LEAF-03# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/101], 00:08:09, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/101], 00:07:20, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 00:08:09, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 00:07:20, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 00:08:09, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 00:07:20, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/200], 00:08:09, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/200], 00:08:09, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/200], 00:07:20, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/200], 00:07:20, ospf-UNDERLAY, intra

```
----

