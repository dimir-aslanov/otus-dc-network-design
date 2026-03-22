# Домашнее задание
Underlay. eBGP
## Цель:
Настроить eBGP для Underlay сети.

---
# Схема
<img width="449" height="371" alt="image" src="https://github.com/user-attachments/assets/9125a61a-93fb-4a3c-a830-00f7c96b8eab" />

---

## IP-план

### SPINE-01

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-01 | Lo0 (loopback1)     | 10.0.1.1   | /32 |
| SPINE-01 | Eth1/1 → LEAF-01    | 10.2.1.0   | /31 |
| SPINE-01 | Eth1/2 → LEAF-02    | 10.2.1.2   | /31 |
| SPINE-01 | Eth1/3 → LEAF-03    | 10.2.1.4   | /31 |

---

### SPINE-02

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-02 | Lo0 (loopback1)     | 10.0.2.1   | /32 |
| SPINE-02 | Eth1/1 → LEAF-01    | 10.2.2.0   | /31 |
| SPINE-02 | Eth1/2 → LEAF-02    | 10.2.2.2   | /31 |
| SPINE-02 | Eth1/3 → LEAF-03    | 10.2.2.4   | /31 |

---

### LEAF-01

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-01 | Lo0 (loopback1)     | 10.0.101.1 | /32 |
| LEAF-01 | Lo1 (loopback2)     | 10.1.101.1 | /32 |
| LEAF-01 | Eth1/1 → SPINE-01   | 10.2.1.1   | /31 |
| LEAF-01 | Eth1/2 → SPINE-02   | 10.2.2.1   | /31 |

---

### LEAF-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-02 | Lo0 (loopback1)     | 10.0.102.1 | /32 |
| LEAF-02 | Lo1 (loopback2)     | 10.1.102.1 | /32 |
| LEAF-02 | Eth1/1 → SPINE-01   | 10.2.1.3   | /31 |
| LEAF-02 | Eth1/2 → SPINE-02   | 10.2.2.3   | /31 |

---

### LEAF-03

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-03 | Lo0 (loopback1)     | 10.0.103.1 | /32 |
| LEAF-03 | Lo1 (loopback2)     | 10.1.103.1 | /32 |
| LEAF-03 | Eth1/1 → SPINE-01   | 10.2.1.5   | /31 |
| LEAF-03 | Eth1/2 → SPINE-02   | 10.2.2.5   | /31 |

---

# Конфигурация eBGP
## SPINE-01
```text
configure terminal

feature bfd
feature bgp
system jumbomtu 9216

route-map RM_Leaf_BGP permit 10
  match as-number 65100-65200

interface Ethernet1/1
  description to-LEAF-01_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.0/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  description to-LEAF-02_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.2/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  description to-LEAF-03_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.4/31
  no ipv6 redirects
  no shutdown

interface loopback0
  description Router-ID
  ip address 10.0.1.1/32

router bgp 65000
  router-id 10.0.1.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  maxas-limit 50
  log-neighbor-changes
  address-family ipv4 unicast
    maximum-paths 10
  neighbor 10.2.1.0/24 remote-as route-map RM_Leaf_BGP
    bfd
    address-family ipv4 unicast

```
## SPINE-02
```text
```text
configure terminal

feature bfd
feature bgp
system jumbomtu 9216

route-map RM_Leaf_BGP permit 10
  match as-number 65100-65200

interface Ethernet1/1
  description to-LEAF-01_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.0/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  description to-LEAF-02_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.2/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/3
  description to-LEAF-03_Eth1/1
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.4/31
  no ipv6 redirects
  no shutdown

interface loopback0
  description Router-ID
  ip address 10.0.2.1/32

router bgp 65000
  router-id 10.0.2.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  maxas-limit 50
  log-neighbor-changes
  address-family ipv4 unicast
    maximum-paths 10
  neighbor 10.2.2.0/24 remote-as route-map RM_Leaf_BGP
    bfd
    address-family ipv4 unicast

```
## LEAF-01
```text

configure terminal

feature bfd
feature bgp
system jumbomtu 9216

route-map RM_REDIS_CON permit 10
  match interface loopback0 loopback1

interface Ethernet1/1
  description TO_SPINE-01
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.1/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  description TO_SPINE-02
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.1/31
  no ipv6 redirects
  no shutdown

interface loopback0
  description UNDERLAY_LOOPBACK
  ip address 10.0.101.1/32

interface loopback1
  description OVERLAY_LOOPBACK
  ip address 10.1.101.1/32

router bgp 65101
  router-id 10.0.101.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  maxas-limit 50
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map RM_REDIS_CON
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.2.1.0
    inherit peer SPINES
  neighbor 10.2.2.0
    inherit peer SPINES


```
## LEAF-02
```text

configure terminal

feature bfd
feature bgp
system jumbomtu 9216

route-map RM_REDIS_CON permit 10
  match interface loopback0 loopback1

interface Ethernet1/1
  description TO_SPINE-01
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.3/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  description TO_SPINE-02
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.3/31
  no ipv6 redirects
  no shutdown

interface loopback0
  description UNDERLAY_LOOPBACK
  ip address 10.0.102.1/32

interface loopback1
  description OVERLAY_LOOPBACK
  ip address 10.1.102.1/32

router bgp 65101
  router-id 10.0.102.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  maxas-limit 50
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map RM_REDIS_CON
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.2.1.2
    inherit peer SPINES
  neighbor 10.2.2.2
    inherit peer SPINES

```
## LEAF-03
```text

configure terminal

feature bfd
feature bgp
system jumbomtu 9216

route-map RM_REDIS_CON permit 10
  match interface loopback0 loopback1

interface Ethernet1/1
  description TO_SPINE-01
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.1.5/31
  no ipv6 redirects
  no shutdown

interface Ethernet1/2
  description TO_SPINE-02
  no switchport
  mtu 9216
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no ip redirects
  ip address 10.2.2.5/31
  no ipv6 redirects
  no shutdown

interface loopback0
  description UNDERLAY_LOOPBACK
  ip address 10.0.103.1/32

interface loopback1
  description OVERLAY_LOOPBACK
  ip address 10.1.103.1/32

router bgp 65101
  router-id 10.0.103.1
  bestpath as-path multipath-relax
  reconnect-interval 12
  maxas-limit 50
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map RM_REDIS_CON
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65000
    timers 3 9
    address-family ipv4 unicast
  neighbor 10.2.1.4
    inherit peer SPINES
  neighbor 10.2.2.4
    inherit peer SPINES

```
---
# Что не получилось в лабе
### BFD - не работает в EVE NG

Нашёл упоминание в документации что BFD не поддерживается.
Bidirectional Forwarding Detection (BFD) is generally unsupported on the virtual Cisco Nexus 9000v (N9Kv) platform, including in EVE-NG simulation. Virtual TCAM/data plane lacks the required hardware acceleration.


```text

LEAF-01(config-route-map)# sh bfd neighbors
OurAddr	  NeighAddr 	LD/RD	RH/RS	Holdown (mult)	State	Int	Vrf	Type
10.2.1.1	10.2.1.0	1090519041/0	Down	N/A(3)	Down	Eth1/1	default	SH
10.2.2.1	10.2.2.0	1090519042/0	Down	N/A(3)	Down	Eth1/2	default	SH


SPINE-01(config-if-range)# sh bfd neighbors
OurAddr	NeighAddr	LD/RD	RH/RS	Holdown (mult)	State	Int	Vrf	Type
10.2.1.0	10.2.1.1	1090519041/0	Down	N/A(3)	Down	Eth1/1	default	SH
10.2.1.2	10.2.1.3	1090519042/0	Down	N/A(3)	Down	Eth1/2	default	SH
10.2.1.4	10.2.1.5	1090519043/0	Down	N/A(3)	Down	Eth1/3	default	SH

```
В детализации BFD вижу на всех коммутаторах что пакеты отправляются но не приходят.  
Rx Count: 0, 
Tx Count: 750

# Вывод CLI

## LEAF-01

```text

LEAF-01(config-route-map)# sh  ip route bGp-65101
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, [20/0], 00:19:38, bgp-65101, external, tag 65000
    *via 10.2.2.0, [20/0], 00:22:32, bgp-65101, external, tag 65000
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, [20/0], 00:21:12, bgp-65101, external, tag 65000
    *via 10.2.2.0, [20/0], 00:21:57, bgp-65101, external, tag 65000
10.1.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, [20/0], 00:19:38, bgp-65101, external, tag 65000
    *via 10.2.2.0, [20/0], 00:20:19, bgp-65101, external, tag 65000
10.1.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, [20/0], 00:20:08, bgp-65101, external, tag 65000
    *via 10.2.2.0, [20/0], 00:20:08, bgp-65101, external, tag 65000

LEAF-01(config-route-map)# sh ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.101.1, local AS number 65101
BGP table version is 24, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 10 paths using 1944 bytes of memory
BGP attribute entries [3/516], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.0        4 65000     537     534       24    0    0 00:20:34 4
10.2.2.0        4 65000     526     524       24    0    0 00:20:13 4

```

## LEAF-02
```text
LEAF-02(config-route-map)# sh ip route bgp-65102
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [20/0], 00:24:25, bgp-65102, external, tag 65000
    *via 10.2.2.2, [20/0], 00:20:58, bgp-65102, external, tag 65000
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [20/0], 00:23:26, bgp-65102, external, tag 65000
    *via 10.2.2.2, [20/0], 00:21:00, bgp-65102, external, tag 65000
10.1.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [20/0], 00:22:14, bgp-65102, external, tag 65000
    *via 10.2.2.2, [20/0], 00:20:58, bgp-65102, external, tag 65000
10.1.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, [20/0], 00:21:48, bgp-65102, external, tag 65000
    *via 10.2.2.2, [20/0], 00:21:00, bgp-65102, external, tag 65000

LEAF-02(config-route-map)# sh ip bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.102.1, local AS number 65102
BGP table version is 24, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 10 paths using 1944 bytes of memory
BGP attribute entries [3/516], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.2        4 65000     493     489       24    0    0 00:21:33 4
10.2.2.2        4 65000     488     485       24    0    0 00:21:18 4

```

## LEAF-03
```text
LEAF-03(config-route-map)# sh ip route bgp-65103
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [20/0], 00:28:09, bgp-65103, external, tag 65000
    *via 10.2.2.4, [20/0], 00:28:19, bgp-65103, external, tag 65000
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [20/0], 00:26:00, bgp-65103, external, tag 65000
    *via 10.2.2.4, [20/0], 00:27:17, bgp-65103, external, tag 65000
10.1.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [20/0], 00:26:56, bgp-65103, external, tag 65000
    *via 10.2.2.4, [20/0], 00:26:56, bgp-65103, external, tag 65000
10.1.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, [20/0], 00:26:00, bgp-65103, external, tag 65000
    *via 10.2.2.4, [20/0], 00:26:41, bgp-65103, external, tag 65000

LEAF-03(config-route-map)# sh ip  bgp summary
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.103.1, local AS number 65103
BGP table version is 22, IPv4 Unicast config peers 2, capable peers 2
6 network entries and 10 paths using 1944 bytes of memory
BGP attribute entries [3/516], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.2.1.4        4 65000     576     573       22    0    0 00:26:44 4
10.2.2.4        4 65000     579     577       22    0    0 00:26:23 4
```
