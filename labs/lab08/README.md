# Домашнее задание
VxLAN. Routing.
## Цель:
Реализовать передачу суммарных префиксов через EVPN route-type 5.

---
# Схема

<img width="739" height="344" alt="image" src="https://github.com/user-attachments/assets/17d4e75c-54e4-469a-9f83-e5ebd4013b1f" />

---

## IP-план

### SPINE-01

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-01 | Lo0 (loopback1)     | 10.0.1.1   | /32 |
| SPINE-01 | Eth1/1 → LEAF-01    | 10.2.1.0   | /31 |
| SPINE-01 | Eth1/2 → LEAF-02    | 10.2.1.2   | /31 |
| SPINE-01 | Eth1/3 → LEAF-03    | 10.2.1.4   | /31 |
| SPINE-01 | Eth1/4 → LEAF-04    | 10.2.1.6   | /31 |

---

### SPINE-02

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-02 | Lo0 (loopback1)     | 10.0.2.1   | /32 |
| SPINE-02 | Eth1/1 → LEAF-01    | 10.2.2.0   | /31 |
| SPINE-02 | Eth1/2 → LEAF-02    | 10.2.2.2   | /31 |
| SPINE-02 | Eth1/3 → LEAF-03    | 10.2.2.4   | /31 |
| SPINE-02 | Eth1/4 → LEAF-04    | 10.2.2.6   | /31 |


---

### LEAF-01

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-01 | Lo0 (loopback1)     | 10.0.101.1 | /32 |
| LEAF-01 | Lo1 (loopback2)     | 10.1.101.1 (10.1.100.1) | /32 |
| LEAF-01 | Eth1/1 → SPINE-01   | 10.2.1.1   | /31 |
| LEAF-01 | Eth1/2 → SPINE-02   | 10.2.2.1   | /31 |
| LEAF-01 | VLAN10  (OTUS-VRF1)            | 172.25.81.1| /24 |
| LEAF-01 | VLAN20      (OTUS-VRF2)        | 172.25.82.1| /24 |
| LEAF-01 | MGMT                | 10.8.0.1   | /24 |


---

### LEAF-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-02 | Lo0 (loopback1)     | 10.0.102.1 | /32 |
| LEAF-02 | Lo1 (loopback2)     | 10.1.102.1 (10.1.100.1)| /32 |
| LEAF-02 | Eth1/1 → SPINE-01   | 10.2.1.3   | /31 |
| LEAF-02 | Eth1/2 → SPINE-02   | 10.2.2.3   | /31 |
| LEAF-02 | VLAN10  (OTUS-VRF1) | 172.25.81.1| /24 |
| LEAF-02 | VLAN20  (OTUS-VRF2) | 172.25.82.1| /24 |
| LEAF-02 | MGMT                | 10.8.0.2   | /24 |


---

### LEAF-03

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-03 | Lo0 (loopback1)     | 10.0.103.1 | /32 |
| LEAF-03 | Lo1 (loopback2)     | 10.1.103.1 (10.1.100.3) | /32 |
| LEAF-03 | Eth1/1 → SPINE-01   | 10.2.1.5   | /31 |
| LEAF-03 | Eth1/2 → SPINE-02   | 10.2.2.5   | /31 |
| LEAF-03 | VLAN10  (OTUS-VRF1) | 172.25.81.1| /24 |
| LEAF-03 | VLAN20  (OTUS-VRF2) | 172.25.82.1| /24 |
| LEAF-03 | VLAN777 (OTUS-VRF1) | 172.25.99.1| /30 |
| LEAF-03 | VLAN666 (OTUS-VRF2) | 172.25.98.1| /30 |
| LEAF-03 | MGMT                | 10.8.0.3   | /24 |


### LEAF-04

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-04 | Lo0 (loopback1)     | 10.0.104.1 | /32 |
| LEAF-04 | Lo1 (loopback2)     | 10.1.104.1 (10.1.100.3)| /32 |
| LEAF-04 | Eth1/1 → SPINE-01   | 10.2.1.7   | /31 |
| LEAF-04 | Eth1/2 → SPINE-02   | 10.2.2.7   | /31 |
| LEAF-04 | VLAN10 (OTUS-VRF1)  | 172.25.81.1| /24 |
| LEAF-04 | VLAN20 (OTUS-VRF2)  | 172.25.82.1| /24 |
| LEAF-04 | VLAN777 (OTUS-VRF1) | 172.25.99.1| /30 |
| LEAF-04 | VLAN666 (OTUS-VRF2) | 172.25.98.1| /30 |
| LEAF-04 | MGMT                | 10.8.0.4   | /24 |


### HV-01

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| HV-01 | VLAN10 (VRF VLAN-10)    | 172.25.81.5 | /24 |
| HV-01 | VLAN20 (VRF VLAN-20)    | 172.25.82.5| /24 |


### HV-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| HV-01 | VLAN10 (VRF VLAN-10)    | 172.25.81.6 | /24 |
| HV-01 | VLAN20 (VRF VLAN-20)    | 172.25.82.6| /24 |

### FW-01

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| FW-01 | VLAN666   | 172.25.98.2 | /30 |
| FW-01 | VLAN777   | 172.25.99.2| /30 |


---

# Конфигурация (EVPN route-type 5)

## LEAF-01
```text
configure terminal

fabric forwarding anycast-gateway-mac 0000.0000.9999
vlan 1,10,20,888,999
vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 888
  name L3_VNI_02
  vn-segment 10888
vlan 999
  name L3_VNI_01
  vn-segment 10999

key chain OSPF-KEYS
  key 1
    key-string 7 0721245859060b0e464058
    cryptographic-algorithm MD5
vrf context OTUS-VRF1
  vni 10999
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context OTUS-VRF2
  vni 10888
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management

vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.2 source 10.8.0.1
  peer-gateway
  layer3 peer-router
  ip arp synchronize

interface Vlan10
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.81.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.82.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan888
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan999
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description vPC
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10888 associate-vrf
  member vni 10999 associate-vrf

interface Ethernet1/3
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/4
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description HV-01
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  channel-group 10 mode active

interface mgmt0
  vrf member management
  ip address 10.8.0.1/24

interface loopback1
  description VTEP_and_Services
  ip address 10.1.101.1/32
  ip address 10.1.100.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0

router bgp 65101
  router-id 10.0.101.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65999
    update-source loopback0
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 10.0.1.1
    inherit peer SPINES
  neighbor 10.0.2.1
    inherit peer SPINES
```

## LEAF-02
```text
configure terminal

fabric forwarding anycast-gateway-mac 0000.0000.9999
vlan 1,10,20,888,999
vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 888
  name L3_VNI_02
  vn-segment 10888
vlan 999
  name L3_VNI_01
  vn-segment 10999

key chain OSPF-KEYS
  key 1
    key-string 7 0721245859060b0e464058
    cryptographic-algorithm MD5
vrf context OTUS-VRF1
  vni 10999
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context OTUS-VRF2
  vni 10888
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management

vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.1 source 10.8.0.2
  peer-gateway
  layer3 peer-router
  ip arp synchronize

interface Vlan10
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.81.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.82.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan888
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan999
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description vPC
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10888 associate-vrf
  member vni 10999 associate-vrf

interface Ethernet1/3
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/4
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description HV-01
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  channel-group 10 mode active

interface mgmt0
  vrf member management
  ip address 10.8.0.2/24

interface loopback1
  description VTEP_and_Services
  ip address 10.1.102.1/32
  ip address 10.1.100.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0

router bgp 65102
  router-id 10.0.102.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65999
    update-source loopback0
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 10.0.1.1
    inherit peer SPINES
  neighbor 10.0.2.1
    inherit peer SPINES

evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto
```


## LEAF-03
```text
configure terminal

fabric forwarding anycast-gateway-mac 0000.0000.9999
vlan 1,10,20,666,777,888,999
vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 666
  name Transit_to_FW_VRF_VLAN-20
vlan 777
  name Transit_to_FW_VRF_VLAN-10
vlan 888
  name L3_VNI_02
  vn-segment 10888
vlan 999
  name L3_VNI_01
  vn-segment 10999

key chain OSPF-KEYS
  key 1
    key-string 7 0721245859060b0e464058
    cryptographic-algorithm MD5
vrf context OTUS-VRF1
  vni 10999
  ip route 0.0.0.0/0 172.25.99.2
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context OTUS-VRF2
  vni 10888
  ip route 0.0.0.0/0 172.25.98.2
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.4 source 10.8.0.3
  peer-gateway
  layer3 peer-router
  ip arp synchronize

interface Vlan10
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.81.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.82.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan666
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.98.1/30
  no ipv6 redirects

interface Vlan777
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.99.1/30
  no ipv6 redirects

interface Vlan888
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan999
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description vPC
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface port-channel100
  description FW-01
  switchport mode trunk
  switchport trunk allowed vlan 666,777
  vpc 100

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10888 associate-vrf
  member vni 10999 associate-vrf


interface Ethernet1/3
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/4
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description HV-02
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  channel-group 10 mode active

interface Ethernet1/7
  description FW-01
  switchport mode trunk
  switchport trunk allowed vlan 666,777
  channel-group 100 mode active

interface mgmt0
  vrf member management
  ip address 10.8.0.3/24

interface loopback1
  description VTEP_and_Services
  ip address 10.1.103.1/32
  ip address 10.1.100.3/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0

router bgp 65103
  router-id 10.0.103.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65999
    update-source loopback0
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 10.0.1.1
    inherit peer SPINES
  neighbor 10.0.2.1
    inherit peer SPINES
  vrf OTUS-VRF1
    address-family ipv4 unicast
      network 0.0.0.0/0
  vrf OTUS-VRF2
    address-family ipv4 unicast
      network 0.0.0.0/0
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto


```

## LEAF-04
```text
configure terminal

fabric forwarding anycast-gateway-mac 0000.0000.9999
vlan 1,10,20,666,777,888,999
vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 666
  name Transit_to_FW_VRF_VLAN-20
vlan 777
  name Transit_to_FW_VRF_VLAN-10
vlan 888
  name L3_VNI_02
  vn-segment 10888
vlan 999
  name L3_VNI_01
  vn-segment 10999

key chain OSPF-KEYS
  key 1
    key-string 7 0721245859060b0e464058
    cryptographic-algorithm MD5
vrf context OTUS-VRF1
  vni 10999
  ip route 0.0.0.0/0 172.25.99.2
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context OTUS-VRF2
  vni 10888
  ip route 0.0.0.0/0 172.25.98.2
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn
vrf context management
vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.3 source 10.8.0.4
  peer-gateway
  layer3 peer-router
  ip arp synchronize

interface Vlan10
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.81.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.82.1/24
  no ipv6 redirects
  fabric forwarding mode anycast-gateway

interface Vlan666
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip address 172.25.98.1/30
  no ipv6 redirects

interface Vlan777
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip address 172.25.99.1/30
  no ipv6 redirects

interface Vlan888
  no shutdown
  vrf member OTUS-VRF2
  no ip redirects
  ip forward
  no ipv6 redirects

interface Vlan999
  no shutdown
  vrf member OTUS-VRF1
  no ip redirects
  ip forward
  no ipv6 redirects

interface port-channel1
  description vPC
  switchport mode trunk
  spanning-tree port type network
  vpc peer-link

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface port-channel100
  description FW-01
  switchport mode trunk
  switchport trunk allowed vlan 666,777
  vpc 100

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
  member vni 10020
    ingress-replication protocol bgp
  member vni 10888 associate-vrf
  member vni 10999 associate-vrf


interface Ethernet1/3
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/4
  description vPC peer-link
  switchport mode trunk
  channel-group 1 mode active

interface Ethernet1/6
  description HV-02
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  channel-group 10 mode active

interface Ethernet1/7
  description FW-01
  switchport mode trunk
  switchport trunk allowed vlan 666,777
  channel-group 100 mode active

interface mgmt0
  vrf member management
  ip address 10.8.0.4/24

interface loopback1
  description VTEP_and_Services
  ip address 10.1.104.1/32
  ip address 10.1.100.3/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0

router bgp 65104
  router-id 10.0.104.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
  template peer SPINES
    bfd
    remote-as 65999
    update-source loopback0
    ebgp-multihop 2
    timers 3 9
    address-family l2vpn evpn
      send-community
      send-community extended
      rewrite-evpn-rt-asn
  neighbor 10.0.1.1
    inherit peer SPINES
  neighbor 10.0.2.1
    inherit peer SPINES
  vrf OTUS-VRF1
    address-family ipv4 unicast
      network 0.0.0.0/0
  vrf OTUS-VRF2
    address-family ipv4 unicast
      network 0.0.0.0/0
evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto
  vni 10020 l2
    rd auto
    route-target import auto
    route-target export auto

```

---

# Вывод CLI

```
## LEAF-01
```text

LEAF-01(config-if)# sh vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     10,20

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20



LEAF-01(config-if)# sh port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        b - BFD Session Wait
        S - Switched    R - Routed
        U - Up (port-channel)
        p - Up in delay-lacp mode (member)
        M - Not in use. Min-links not met
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
1     Po1(SU)     Eth      LACP      Eth1/3(P)    Eth1/4(P)
10    Po10(SU)    Eth      LACP      Eth1/6(P)

```


## LEAF-02
```text
LEAF-02(config-if)# sh vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20


LEAF-02(config-if)# sh port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        b - BFD Session Wait
        S - Switched    R - Routed
        U - Up (port-channel)
        p - Up in delay-lacp mode (member)
        M - Not in use. Min-links not met
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
1     Po1(SU)     Eth      LACP      Eth1/3(P)    Eth1/4(P)
10    Po10(SU)    Eth      LACP      Eth1/6(P)
LEAF-02(config-if)#


```

## LEAF-03
```text

LEAF-03(config-if)# sh vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20


LEAF-03(config-if)# sh port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        b - BFD Session Wait
        S - Switched    R - Routed
        U - Up (port-channel)
        p - Up in delay-lacp mode (member)
        M - Not in use. Min-links not met
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
1     Po1(SU)     Eth      LACP      Eth1/3(P)    Eth1/4(P)
10    Po10(SU)    Eth      LACP      Eth1/6(P)

```


## LEAF-04
```text

LEAF-04(config-if)# sh vpc  brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 1
Peer Gateway                      : Disabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Disabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20


LEAF-04(config-if)# sh port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        b - BFD Session Wait
        S - Switched    R - Routed
        U - Up (port-channel)
        p - Up in delay-lacp mode (member)
        M - Not in use. Min-links not met
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
1     Po1(SU)     Eth      LACP      Eth1/3(P)    Eth1/4(P)
10    Po10(SU)    Eth      LACP      Eth1/6(P)


```

## HV-01
```text

HV-01# sh ip int brief vrf all

IP Interface Status for VRF "VLAN-10"(3)
Interface            IP Address      Interface Status
Vlan10               172.25.81.5     protocol-up/link-up/admin-up

IP Interface Status for VRF "VLAN-20"(4)
Interface            IP Address      Interface Status
Vlan20               172.25.82.5     protocol-up/link-up/admin-up

HV-01# ping 172.25.81.6 vrf vLAN-10
PING 172.25.81.6 (172.25.81.6): 56 data bytes
36 bytes from 172.25.81.5: Destination Host Unreachable
Request 0 timed out
64 bytes from 172.25.81.6: icmp_seq=1 ttl=254 time=15.515 ms
64 bytes from 172.25.81.6: icmp_seq=2 ttl=254 time=10.607 ms
64 bytes from 172.25.81.6: icmp_seq=3 ttl=254 time=13.146 ms

HV-01# ping 172.25.82.6 vrf vLAN-20
PING 172.25.82.6 (172.25.82.6): 56 data bytes
64 bytes from 172.25.82.6: icmp_seq=0 ttl=254 time=17.959 ms
64 bytes from 172.25.82.6: icmp_seq=1 ttl=254 time=14.832 ms
64 bytes from 172.25.82.6: icmp_seq=2 ttl=254 time=10.832 ms

HV-01# sh port-channel summary
Flags:  D - Down        P - Up in port-channel (members)
        I - Individual  H - Hot-standby (LACP only)
        s - Suspended   r - Module-removed
        b - BFD Session Wait
        S - Switched    R - Routed
        U - Up (port-channel)
        p - Up in delay-lacp mode (member)
        M - Not in use. Min-links not met
--------------------------------------------------------------------------------
Group Port-       Type     Protocol  Member Ports
      Channel
--------------------------------------------------------------------------------
1     Po1(SU)     Eth      LACP      Eth1/1(P)    Eth1/2(P)

```
