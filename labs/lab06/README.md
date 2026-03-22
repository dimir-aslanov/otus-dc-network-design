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
| LEAF-03 | VLAN10              | 172.25.85.1| /24 |
| LEAF-03 | VLAN20              | 172.25.85.1| /24 |


---


# Конфигурация eBGP EVPN VxLAN
## SPINE-01
```text
configure terminal

feature nv overlay
nv overlay evpn
feature bgp

route-map EVPN_NH_UNCHANGED permit 10
  set ip next-hop unchanged

router bgp 65999
  router-id 10.0.1.1
  timers bgp 3 9
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all

  neighbor 10.0.101.1
    remote-as 65101
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn

  neighbor 10.0.102.1
    remote-as 65102
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn

  neighbor 10.0.103.1
    remote-as 65103
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn
```
## SPINE-02
```text
configure terminal

feature nv overlay
nv overlay evpn
feature bgp

route-map EVPN_NH_UNCHANGED permit 10
  set ip next-hop unchanged

router bgp 65999
  router-id 10.0.2.1
  timers bgp 3 9
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10
    retain route-target all

  neighbor 10.0.101.1
    remote-as 65101
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn

  neighbor 10.0.102.1
    remote-as 65102
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn

  neighbor 10.0.103.1
    remote-as 65103
    update-source loopback0
    ebgp-multihop 5
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map EVPN_NH_UNCHANGED out
      rewrite-evpn-rt-asn
```
## LEAF-01
```text
configure terminal

feature vn-segment-vlan-based
feature nv overlay
nv overlay evpn
feature bgp

router bgp 65101
  router-id 10.0.101.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10

  template peer SPINES
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


vlan 10
  name VLAN_10
  vn-segment 10010

evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp



```
## LEAF-02
```text

configure terminal

feature vn-segment-vlan-based
feature nv overlay
nv overlay evpn
feature bgp

router bgp 65102
  router-id 10.0.102.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10

  template peer SPINES
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

vlan 10
  name VLAN_10
  vn-segment 10010

evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp

```
## LEAF-03
```text

configure terminal

feature vn-segment-vlan-based
feature nv overlay
nv overlay evpn
feature bgp

router bgp 65103
  router-id 10.0.103.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  reconnect-interval 10
  log-neighbor-changes
  address-family l2vpn evpn
    maximum-paths 10

  template peer SPINES
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

vlan 10
  name VLAN_10
  vn-segment 10010

evpn
  vni 10010 l2
    rd auto
    route-target import auto
    route-target export auto

interface nve1
  no shutdown
  host-reachability protocol bgp
  source-interface loopback1
  member vni 10010
    ingress-replication protocol bgp
```
---

# Вывод CLI

```
## LEAF-01
```text

LEAF-01(config)# show bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.101.1, local AS number 65101
BGP table version is 78, L2VPN EVPN config peers 2, capable peers 2
12 network entries and 17 paths using 2928 bytes of memory
BGP attribute entries [8/1376], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    1312    1298       78    0    0 01:04:45 5
10.0.2.1        4 65999    1310    1298       78    0    0 01:04:45 5


LEAF-01(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
nve1      10.1.102.1                              Up    CP        01:01:23 n/a

nve1      10.1.103.1                              Up    CP        01:01:57 n/a


LEAF-01(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*   10     0050.7966.6806   dynamic  0         F      F    Eth1/7
C   10     0050.7966.6807   dynamic  0         F      F    nve1(10.1.102.1)
C   10     0050.7966.6808   dynamic  0         F      F    nve1(10.1.103.1)
C   10     0050.7966.6809   dynamic  0         F      F    nve1(10.1.103.1)
G    -     5001.0000.1b08   static   -         F      F    sup-eth1(R)
G   10     5001.0000.1b08   static   -         F      F    sup-eth1(R)


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

