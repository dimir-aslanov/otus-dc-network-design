# Домашнее задание
VXLAN. Multihoming
## Цель:
Настроить отказоустойчивое подключение клиентов с использованием EVPN Multihoming | vPC.

---
# Схема

<img width="709" height="378" alt="image" src="https://github.com/user-attachments/assets/2ebb1020-fc88-4230-8745-25248884d520" />

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
| LEAF-01 | Lo1 (loopback2)     | 10.1.101.1 | /32 |
| LEAF-01 | Eth1/1 → SPINE-01   | 10.2.1.1   | /31 |
| LEAF-01 | Eth1/2 → SPINE-02   | 10.2.2.1   | /31 |
| LEAF-01 | VLAN10              | 172.25.81.1| /24 |
| LEAF-01 | VLAN20              | 172.25.82.1| /24 |
| LEAF-01 | MGMT                | 10.8.0.1   | /24 |


---

### LEAF-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-02 | Lo0 (loopback1)     | 10.0.102.1 | /32 |
| LEAF-02 | Lo1 (loopback2)     | 10.1.102.1 | /32 |
| LEAF-02 | Eth1/1 → SPINE-01   | 10.2.1.3   | /31 |
| LEAF-02 | Eth1/2 → SPINE-02   | 10.2.2.3   | /31 |
| LEAF-02 | VLAN10              | 172.25.81.1| /24 |
| LEAF-02 | VLAN20              | 172.25.82.1| /24 |
| LEAF-02 | MGMT                | 10.8.0.2   | /24 |


---

### LEAF-03

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-03 | Lo0 (loopback1)     | 10.0.103.1 | /32 |
| LEAF-03 | Lo1 (loopback2)     | 10.1.103.1 | /32 |
| LEAF-03 | Eth1/1 → SPINE-01   | 10.2.1.5   | /31 |
| LEAF-03 | Eth1/2 → SPINE-02   | 10.2.2.5   | /31 |
| LEAF-03 | VLAN10              | 172.25.81.1| /24 |
| LEAF-03 | VLAN20              | 172.25.82.1| /24 |
| LEAF-03 | MGMT                | 10.8.0.3   | /24 |


### LEAF-04

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-04 | Lo0 (loopback1)     | 10.0.104.1 | /32 |
| LEAF-04 | Lo1 (loopback2)     | 10.1.104.1 | /32 |
| LEAF-04 | Eth1/1 → SPINE-01   | 10.2.1.7   | /31 |
| LEAF-04 | Eth1/2 → SPINE-02   | 10.2.2.7   | /31 |
| LEAF-04 | VLAN10              | 172.25.81.1| /24 |
| LEAF-04 | VLAN20              | 172.25.82.1| /24 |
| LEAF-04 | MGMT                | 10.8.0.4   | /24 |

---


# Конфигурация vPC

## LEAF-01
```text
configure terminal

feature interface-vlan
feature lacp
feature vpc

interface mgmt0
  vrf member management
  ip address 10.8.0.1/24
  no shutdown

vpc domain 1
  peer-switch
  role priority 10
  peer-keepalive destination 10.8.0.2 source 10.8.0.1

interface ethernet1/3
  channel-group 1 mode active

interface ethernet1/4
  channel-group 1 mode active

interface port-channel1
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc peer-link

interface ethernet1/6
  description HV-01
  channel-group 10 mode active

interface port-channel10
  description HV-01
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface loopback1
  description VTEP_and_Services
  ip address 10.1.101.1/32
  ip address 10.1.102.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0



```
## LEAF-02
```text
configure terminal

feature interface-vlan
feature lacp
feature vpc

interface mgmt0
  vrf member management
  ip address 10.8.0.2/24
  no shutdown

vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.1 source 10.8.0.2

interface ethernet1/3
  description vPC
  channel-group 1 mode active

interface ethernet1/4
  description vPC
  channel-group 1 mode active

interface port-channel1
  description vPC
  switchport
  switchport mode trunk
 switchport trunk allowed vlan 10,20
  vpc peer-link

interface ethernet1/6
  description HV-01
  channel-group 10 mode active

interface port-channel10
  description HV-01
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface loopback1
  description VTEP_and_Services
  ip address 10.1.101.1/32
  ip address 10.1.102.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0


```

## LEAF-03
```text
configure terminal

feature interface-vlan
feature lacp
feature vpc

interface mgmt0
  vrf member management
  ip address 10.8.0.3/24
  no shutdown

vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.4 source 10.8.0.3

interface ethernet1/3
  description vPC
  channel-group 1 mode active

interface ethernet1/4
  description vPC
  channel-group 1 mode active

interface port-channel1
  description vPC
  switchport
  switchport mode trunk
 switchport trunk allowed vlan 10,20
  vpc peer-link

interface ethernet1/6
  description HV-02
  channel-group 10 mode active

interface port-channel10
  description HV-02
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface loopback1
  description VTEP_and_Services
  ip address 10.1.103.1/32
  ip address 10.1.104.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0


```

## LEAF-04
```text
configure terminal

feature interface-vlan
feature lacp
feature vpc

interface mgmt0
  vrf member management
  ip address 10.8.0.4/24
  no shutdown

vpc domain 1
  peer-switch
  role priority 20
  peer-keepalive destination 10.8.0.3 source 10.8.0.4

interface ethernet1/3
  description vPC
  channel-group 1 mode active

interface ethernet1/4
  description vPC
  channel-group 1 mode active

interface port-channel1
  description vPC
  switchport
  switchport mode trunk
 switchport trunk allowed vlan 10,20
  vpc peer-link

interface ethernet1/6
  description HV-02
  channel-group 10 mode active

interface port-channel10
  description HV-02
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 10,20
  vpc 10

interface loopback1
  description VTEP_and_Services
  ip address 10.1.103.1/32
  ip address 10.1.104.1/32 secondary
  ip router ospf UNDERLAY area 0.0.0.0

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

При отключении одного из линков в Po, связь сохраняется.
