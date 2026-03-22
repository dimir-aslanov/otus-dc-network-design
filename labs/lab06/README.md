# Домашнее задание
VxLAN. L3VNI
## Цель:
Настроить маршрутизацию в рамках Overlay между клиентами.

---
# Схема

<img width="601" height="361" alt="image" src="https://github.com/user-attachments/assets/76cbde99-d197-4416-a650-d83f1f2554f6" />

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
| LEAF-01 | VLAN10              | 172.25.81.1| /24 |
| LEAF-01 | VLAN20              | 172.25.82.1| /24 |


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


---


# Конфигурация eBGP EVPN VxLAN

## LEAF-01
```text
configure terminal

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 999
  name L3_VNI
  vn-segment 10999

fabric forwarding anycast-gateway-mac 0000.0000.9999

vrf context OTUS
  vni 10999
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan10
  no shutdown
  vrf member OTUS
  ip address 172.25.81.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS
  ip address 172.25.82.1/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member OTUS
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
    suppress-arp
  member vni 10020
    ingress-replication protocol bgp
     suppress-arp
  member vni 10999 associate-vrf

interface Ethernet1/6
  description VM-05
  switchport access vlan 20

interface Ethernet1/7
  description VM-01
  switchport access vlan 10

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
## LEAF-02
```text

configure terminal

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 999
  name L3_VNI
  vn-segment 10999

fabric forwarding anycast-gateway-mac 0000.0000.9999

vrf context OTUS
  vni 10999
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan10
  no shutdown
  vrf member OTUS
  ip address 172.25.81.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS
  ip address 172.25.82.1/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member OTUS
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
    suppress-arp
  member vni 10020
    ingress-replication protocol bgp
     suppress-arp
  member vni 10999 associate-vrf

interface Ethernet1/7
  description VM-02
  switchport access vlan 20

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

nv overlay evpn
feature ospf
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

vlan 10
  name VLAN10
  vn-segment 10010
vlan 20
  name VLAN20
  vn-segment 10020
vlan 999
  name L3_VNI
  vn-segment 10999

fabric forwarding anycast-gateway-mac 0000.0000.9999

vrf context OTUS
  vni 10999
  rd auto
  address-family ipv4 unicast
    route-target both auto
    route-target both auto evpn

interface Vlan10
  no shutdown
  vrf member OTUS
  ip address 172.25.81.1/24
  fabric forwarding mode anycast-gateway

interface Vlan20
  no shutdown
  vrf member OTUS
  ip address 172.25.82.1/24
  fabric forwarding mode anycast-gateway

interface Vlan999
  no shutdown
  vrf member OTUS
  ip forward

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
    suppress-arp
  member vni 10020
    ingress-replication protocol bgp
     suppress-arp
  member vni 10999 associate-vrf

interface Ethernet1/6
  description VM-03
  switchport access vlan 10

interface Ethernet1/7
  description VM-04
  switchport access vlan 20

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

LEAF-01(config-if)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*   10     0050.7966.6806   dynamic  0         F      F    Eth1/7
C   10     0050.7966.6808   dynamic  0         F      F    nve1(10.1.103.1)
C   20     0050.7966.6807   dynamic  0         F      F    nve1(10.1.102.1)
*   20     0050.7966.680b   dynamic  0         F      F    Eth1/6
*  999     5001.0000.1b08   static   -         F      F    Vlan999
*  999     5004.0000.1b08   static   -         F      F    nve1(10.1.103.1)
*  999     5005.0000.1b08   static   -         F      F    nve1(10.1.102.1)
G    -     0000.0000.9999   static   -         F      F    sup-eth1(R)
G    -     5001.0000.1b08   static   -         F      F    sup-eth1(R)
G   10     5001.0000.1b08   static   -         F      F    sup-eth1(R)
G   20     5001.0000.1b08   static   -         F      F    sup-eth1(R)
G  999     5001.0000.1b08   static   -         F      F    sup-eth1(R)


LEAF-01(config-if)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.102.1                              Up    CP        14:02:47 5005.0000.1b08
nve1      10.1.103.1                              Up    CP        14:02:47 5004.0000.1b08


LEAF-01(config-if)# sh vxlan interface
connect localhost:56000 failed: Connection refused
Interface       Vlan    VPL Ifindex     LTL             HW VP
=========       ====    ===========     ===             =====
Eth1/6          20      0x530147fb      0x1801          2050
Eth1/7          10      0x5300a7fa      0x1802          2051


LEAF-01(config-if)# sh nve vrf
VRF-Name     VNI        Interface Gateway-MAC
------------ ---------- --------- -----------------
OTUS         10999      nve1      5001.0000.1b08

```

## LEAF-02
```text

LEAF-02# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     0050.7966.6806   dynamic  0         F      F    nve1(10.1.101.1)
C   10     0050.7966.6808   dynamic  0         F      F    nve1(10.1.103.1)
*   20     0050.7966.6807   dynamic  0         F      F    Eth1/7
C   20     0050.7966.6809   dynamic  0         F      F    nve1(10.1.103.1)
C   20     0050.7966.680b   dynamic  0         F      F    nve1(10.1.101.1)
*  999     5001.0000.1b08   static   -         F      F    nve1(10.1.101.1)
*  999     5004.0000.1b08   static   -         F      F    nve1(10.1.103.1)
*  999     5005.0000.1b08   static   -         F      F    Vlan999
G    -     0000.0000.9999   static   -         F      F    sup-eth1(R)
G    -     5005.0000.1b08   static   -         F      F    sup-eth1(R)
G   10     5005.0000.1b08   static   -         F      F    sup-eth1(R)
G   20     5005.0000.1b08   static   -         F      F    sup-eth1(R)
G  999     5005.0000.1b08   static   -         F      F    sup-eth1(R)

LEAF-02# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
nve1      10.1.101.1                              Up    CP        14:23:24 5001.0000.1b08
nve1      10.1.103.1                              Up    CP        16:10:16 5004.0000.1b08


LEAF-02# sh vxlan interface
connect localhost:56000 failed: Connection refused
Interface       Vlan    VPL Ifindex     LTL             HW VP
=========       ====    ===========     ===             =====
Eth1/7          20      0x530147fa      0x1801          2050

LEAF-02# sh nve vrf
VRF-Name     VNI        Interface Gateway-MAC
------------ ---------- --------- -----------------
OTUS         10999      nve1      5005.0000.1b08


```



## LEAF-03
```text

LEAF-03(config-evpn-evi)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     0050.7966.6806   dynamic  0         F      F    nve1(10.1.101.1)
*   10     0050.7966.6808   dynamic  0         F      F    Eth1/6
C   20     0050.7966.6807   dynamic  0         F      F    nve1(10.1.102.1)
*   20     0050.7966.6809   dynamic  0         F      F    Eth1/7
C   20     0050.7966.680b   dynamic  0         F      F    nve1(10.1.101.1)
*  999     5001.0000.1b08   static   -         F      F    nve1(10.1.101.1)
*  999     5004.0000.1b08   static   -         F      F    Vlan999
*  999     5005.0000.1b08   static   -         F      F    nve1(10.1.102.1)
G    -     0000.0000.9999   static   -         F      F    sup-eth1(R)
G    -     5004.0000.1b08   static   -         F      F    sup-eth1(R)
G   10     5004.0000.1b08   static   -         F      F    sup-eth1(R)
G   20     5004.0000.1b08   static   -         F      F    sup-eth1(R)
G  999     5004.0000.1b08   static   -         F      F    sup-eth1(R)

LEAF-03(config-evpn-evi)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----

nve1      10.1.101.1                              Up    CP        14:27:10 5001.0000.1b08
nve1      10.1.102.1                              Up    CP        14:28:59 5005.0000.1b08


LEAF-03(config-evpn-evi)# sh vxlan interface
connect localhost:56000 failed: Connection refused
Interface       Vlan    VPL Ifindex     LTL             HW VP
=========       ====    ===========     ===             =====
Eth1/6          10      0x5300a7fb      0x1801          2050
Eth1/7          20      0x530147fa      0x1802          2051


LEAF-03(config-evpn-evi)# sh nve vrf
VRF-Name     VNI        Interface Gateway-MAC
------------ ---------- --------- -----------------
OTUS         10999      nve1      5004.0000.1b08

```

## VMs
```text

VM-03> show
NAME   IP/MASK              GATEWAY                            
VM-03  172.25.81.6/24       172.25.81.1

VM-03> ping 172.25.81.5
84 bytes from 172.25.81.5 icmp_seq=1 ttl=64 time=6.577 ms
84 bytes from 172.25.81.5 icmp_seq=2 ttl=64 time=7.250 ms
84 bytes from 172.25.81.5 icmp_seq=3 ttl=64 time=6.386 ms

VM-03> ping 172.25.82.5
84 bytes from 172.25.82.5 icmp_seq=1 ttl=62 time=9.118 ms
84 bytes from 172.25.82.5 icmp_seq=2 ttl=62 time=7.903 ms
84 bytes from 172.25.82.5 icmp_seq=3 ttl=62 time=8.809 ms

VM-03> ping 172.25.82.6
84 bytes from 172.25.82.6 icmp_seq=1 ttl=62 time=15.933 ms
84 bytes from 172.25.82.6 icmp_seq=2 ttl=62 time=8.182 ms
84 bytes from 172.25.82.6 icmp_seq=3 ttl=62 time=8.976 ms

VM-03> ping 172.25.82.7
84 bytes from 172.25.82.7 icmp_seq=1 ttl=63 time=4.992 ms
84 bytes from 172.25.82.7 icmp_seq=2 ttl=63 time=2.574 ms
84 bytes from 172.25.82.7 icmp_seq=3 ttl=63 time=2.137 ms

VM-03> show arp
00:50:79:66:68:06  172.25.81.5 expires in 24 seconds
00:00:00:00:99:99  172.25.81.1 expires in 88 seconds


VM-05> show
NAME   IP/MASK              GATEWAY                             
VM-05  172.25.82.6/24       172.25.82.1

VM-05> ping 172.25.82.7
84 bytes from 172.25.82.7 icmp_seq=1 ttl=64 time=8.413 ms
84 bytes from 172.25.82.7 icmp_seq=2 ttl=64 time=7.481 ms
84 bytes from 172.25.82.7 icmp_seq=3 ttl=64 time=6.250 ms

VM-05> ping 172.25.82.5
84 bytes from 172.25.82.5 icmp_seq=1 ttl=64 time=7.634 ms
84 bytes from 172.25.82.5 icmp_seq=2 ttl=64 time=5.925 ms
84 bytes from 172.25.82.5 icmp_seq=3 ttl=64 time=6.013 ms

VM-05> ping 172.25.81.6
84 bytes from 172.25.81.6 icmp_seq=1 ttl=62 time=8.711 ms
84 bytes from 172.25.81.6 icmp_seq=2 ttl=62 time=7.299 ms
84 bytes from 172.25.81.6 icmp_seq=3 ttl=62 time=6.592 ms

VM-05> ping 172.25.81.5
84 bytes from 172.25.81.5 icmp_seq=1 ttl=63 time=3.894 ms
84 bytes from 172.25.81.5 icmp_seq=2 ttl=63 time=2.000 ms
84 bytes from 172.25.81.5 icmp_seq=3 ttl=63 time=5.230 ms

VM-05> arp
00:50:79:66:68:07  172.25.82.5 expires in 42 seconds
00:00:00:00:99:99  172.25.82.1 expires in 75 seconds
00:50:79:66:68:09  172.25.82.7 expires in 37 seconds


```

