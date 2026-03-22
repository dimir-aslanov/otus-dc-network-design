<img width="151" height="46" alt="image" src="https://github.com/user-attachments/assets/c31f7928-7abd-4ec6-9f93-22129caf4eaf" /><img width="601" height="361" alt="image" src="https://github.com/user-attachments/assets/104b412e-4ff5-4cef-9484-c14a134a869f" /># Домашнее задание
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

```


## LEAF-02
```text
LEAF-02(config)# sh bgp l2vpn evpn  summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.102.1, local AS number 65102
BGP table version is 81, L2VPN EVPN config peers 2, capable peers 2
12 network entries and 17 paths using 2928 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    1800    1785       81    0    0 01:29:13 5
10.0.2.1        4 65999    1806    1787       81    0    0 01:29:18 5

LEAF-02(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Router-Mac
--------- --------------------------------------  ----- --------- -------- -----------------
nve1      10.1.101.1                              Up    CP        01:45:45 n/a
nve1      10.1.103.1                              Up    CP        01:29:36 n/a

LEAF-02(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     0050.7966.6806   dynamic  0         F      F    nve1(10.1.101.1)
*   10     0050.7966.6807   dynamic  0         F      F    Eth1/7
C   10     0050.7966.6808   dynamic  0         F      F    nve1(10.1.103.1)
C   10     0050.7966.6809   dynamic  0         F      F    nve1(10.1.103.1)
G    -     5005.0000.1b08   static   -         F      F    sup-eth1(R)
G   10     5005.0000.1b08   static   -         F      F    sup-eth1(R)

```
## LEAF-03
```text

LEAF-03# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.103.1, local AS number 65103
BGP table version is 75, L2VPN EVPN config peers 2, capable peers 2
11 network entries and 15 paths using 2684 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    1799    1785       75    0    0 01:29:12 4
10.0.2.1        4 65999    1798    1784       75    0    0 01:29:10 4

LEAF-03# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.101.1                              Up    CP        01:29:26 n/a

nve1      10.1.102.1                              Up    CP        01:28:52 n/a


LEAF-03(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     0050.7966.6806   dynamic  0         F      F    nve1(10.1.101.1)
C   10     0050.7966.6807   dynamic  0         F      F    nve1(10.1.102.1)
*   10     0050.7966.6808   dynamic  0         F      F    Eth1/6
*   10     0050.7966.6809   dynamic  0         F      F    Eth1/7
G    -     5004.0000.1b08   static   -         F      F    sup-eth1(R)

```

## VMs
```text

VM-01> ping 172.25.80.10
84 bytes from 172.25.80.10 icmp_seq=1 ttl=64 time=6.316 ms
84 bytes from 172.25.80.10 icmp_seq=2 ttl=64 time=6.214 ms
84 bytes from 172.25.80.10 icmp_seq=3 ttl=64 time=7.021 ms

VM-02> ping 172.25.80.6
84 bytes from 172.25.80.6 icmp_seq=1 ttl=64 time=7.111 ms
84 bytes from 172.25.80.6 icmp_seq=2 ttl=64 time=7.601 ms

VM-02> ping 172.25.80.12
84 bytes from 172.25.80.12 icmp_seq=1 ttl=64 time=6.086 ms
84 bytes from 172.25.80.12 icmp_seq=2 ttl=64 time=6.194 ms

VM-04> ping 172.25.80.6
84 bytes from 172.25.80.6 icmp_seq=1 ttl=64 time=5.752 ms
84 bytes from 172.25.80.6 icmp_seq=2 ttl=64 time=6.004 ms
84 bytes from 172.25.80.6 icmp_seq=3 ttl=64 time=9.964 ms
84 bytes from 172.25.80.6 icmp_seq=4 ttl=64 time=6.838 ms

LEAF-01(config)# sh ip arp

IP ARP Table for context default
Total number of entries: 7
Address         Age       MAC Address     Interface       Flags
10.2.1.0        00:02:31  5003.0000.1b08  Ethernet1/1
10.2.2.0        00:02:15  5002.0000.1b08  Ethernet1/2
172.25.80.6     00:08:30  0050.7966.6806  Vlan10
172.25.80.9     00:11:22  5005.0000.1b08  Vlan10
172.25.80.10    00:08:30  0050.7966.6807  Vlan10
172.25.80.11    00:08:30  0050.7966.6808  Vlan10
172.25.80.12    00:08:30  0050.7966.6809  Vlan10

```

