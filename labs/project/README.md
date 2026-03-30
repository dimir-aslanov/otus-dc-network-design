# Тема проекта
Проектирование сетевой фабрики на основе VxLAN EVPN

## Цель:
Спроектировать и реализовать отказоустойчивую L2-сетевую фабрику ЦОД на базе VXLAN EVPN с использованием Cisco Nexus 9000, обеспечивающую масштабируемое расширение L2-сегментов и централизованную маршрутизацию и фильтрацию трафика на внешнем межсетевом экране.

### Задачи:

1. Спроектировать архитектуру фабрики:
- Разработать топологию Leaf–Spine на базе Cisco Nexus 9000
- Обеспечить отказоустойчивость (ECMP, vPC) и схему подключения FW

2. Реализовать Underlay-сеть:
- Настроить IP-фабрику на базе OSPF
- Обеспечить IP-доступность между VTEP (Loopback-интерфейсы)

3. Реализовать Overlay VXLAN EVPN (L2):
- Настроить VXLAN EVPN с использованием BGP EVPN
- Реализовать растяжение VLAN (L2VNI) и распространение MAC-адресов

4. Интегрировать межсетевой экран
- Обеспечить терминацию всех VLAN на FW 
- Реализовать прохождение всего межсегментного трафика через FW

5. Провести тестирование и валидацию
- Проверить L2-связность и работу через FW
- Подтвердить отказоустойчивость фабрики при сбоях



---
# Схема

<img width="1113" height="524" alt="image" src="https://github.com/user-attachments/assets/d005bfc0-5e5a-4c10-b167-d934684fddbe" />

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
| LEAF-01 | MGMT                | 10.8.0.1   | /24 |


---

### LEAF-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-02 | Lo0 (loopback1)     | 10.0.102.1 | /32 |
| LEAF-02 | Lo1 (loopback2)     | 10.1.102.1 (10.1.100.1)| /32 |
| LEAF-02 | Eth1/1 → SPINE-01   | 10.2.1.3   | /31 |
| LEAF-02 | Eth1/2 → SPINE-02   | 10.2.2.3   | /31 |
| LEAF-02 | MGMT                | 10.8.0.2   | /24 |


---

### LEAF-03

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-03 | Lo0 (loopback1)     | 10.0.103.1 | /32 |
| LEAF-03 | Lo1 (loopback2)     | 10.1.103.1 (10.1.100.3) | /32 |
| LEAF-03 | Eth1/1 → SPINE-01   | 10.2.1.5   | /31 |
| LEAF-03 | Eth1/2 → SPINE-02   | 10.2.2.5   | /31 |
| LEAF-03 | MGMT                | 10.8.0.3   | /24 |


### LEAF-04

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-04 | Lo0 (loopback1)     | 10.0.104.1 | /32 |
| LEAF-04 | Lo1 (loopback2)     | 10.1.104.1 (10.1.100.3)| /32 |
| LEAF-04 | Eth1/1 → SPINE-01   | 10.2.1.7   | /31 |
| LEAF-04 | Eth1/2 → SPINE-02   | 10.2.2.7   | /31 |
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
| FW-01 | VLAN10 (OTUS-VRF1)   | 172.25.81.1 | /24 |
| FW-01 | VLAN20 (OTUS-VRF2)   | 172.25.82.1| /24 |


---

# Конфигурации

 - [SPINE-01](configs/SPINE-01.txt)
 - [SPINE-02](configs/SPINE-02.txt)
 - [LEAF-01](configs/LEAF-01.txt)
 - [LEAF-02](configs/LEAF-02.txt)
 - [LEAF-03](configs/LEAF-03.txt)
 - [FW-01](configs/FW-01.txt)
 - [HV-01](configs/HV-01.txt)
 - [HV-02](configs/HV-02.txt)

# Вывод CLI


## LEAF-01
```text
LEAF-01(config)# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  10.0.101.1      protocol-up/link-up/admin-up
Lo1                  10.1.101.1      protocol-up/link-up/admin-up
Eth1/1               10.2.1.1        protocol-up/link-up/admin-up
Eth1/2               10.2.2.1        protocol-up/link-up/admin-up

```
```text
LEAF-01(config)# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          01:28:10 10.2.1.0        Eth1/1
 10.0.2.1          1 FULL/ -          01:28:48 10.2.2.0        Eth1/2
```
```text
LEAF-01(config)# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/101], 01:28:28, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/101], 01:29:01, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:23, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:27, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.0.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:23, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.1.100.3/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:27, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.1.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:23, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.1.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:27, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.1.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.0, Eth1/1, [110/201], 01:28:23, ospf-UNDERLAY, intra
    *via 10.2.2.0, Eth1/2, [110/201], 01:29:01, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/200], 01:28:28, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/200], 01:28:28, ospf-UNDERLAY, intra
10.2.1.6/31, ubest/mbest: 1/0
    *via 10.2.1.0, Eth1/1, [110/200], 01:28:28, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/200], 01:29:01, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/200], 01:29:01, ospf-UNDERLAY, intra
10.2.2.6/31, ubest/mbest: 1/0
    *via 10.2.2.0, Eth1/2, [110/200], 01:29:01, ospf-UNDERLAY, intra
```
```text
LEAF-01(config)# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.101.1, local AS number 65101
BGP table version is 200, L2VPN EVPN config peers 2, capable peers 2
22 network entries and 40 paths using 6088 bytes of memory
BGP attribute entries [16/2752], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    1839    1786      200    0    0 00:45:12 12
10.0.2.1        4 65999    1852    1797      200    0    0 00:45:12 12

```
```text
LEAF-01(config)# sh bgp l2vpn evpn

BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 200, Local Router ID is 10.0.101.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.0.101.1:32777    (L2VNI 10010)
*>l[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                        100      32768 i
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65104 i
*>l[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                        100      32768 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.101.1:32787    (L2VNI 10020)
*>l[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                        100      32768 i
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65104 i
*>l[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                        100      32768 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.103.1:32777
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i
*>e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
* e                   10.1.100.3                                     0 65999 65103 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.103.1:32787
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i
*>e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
* e                   10.1.100.3                                     0 65999 65103 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.104.1:32777
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i

Route Distinguisher: 10.0.104.1:32787
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i


```
```text
LEAF-01(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
+   10     5006.0000.1b08   dynamic  0         F      F    Po10
C   10     5008.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
C   10     500a.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
+   20     5006.0000.1b08   dynamic  0         F      F    Po10
C   20     5008.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
C   20     500a.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
G    -     5001.0000.1b08   static   -         F      F    sup-eth1(R)
G    -     5001.0000.1b08   static   -         F      F    Lo0(R) (Lo0)

```
```text
LEAF-01(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.100.3                              Up    CP        00:46:54 n/a
```
```text
LEAF-01(config)# sh vpc brief
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
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20

```



## LEAF-02
```text
LEAF-02(config)# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  10.0.102.1      protocol-up/link-up/admin-up
Lo1                  10.1.102.1      protocol-up/link-up/admin-up
Eth1/1               10.2.1.3        protocol-up/link-up/admin-up
Eth1/2               10.2.2.3        protocol-up/link-up/admin-up
```
```text
LEAF-02(config)# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          01:34:29 10.2.1.2        Eth1/1
 10.0.2.1          1 FULL/ -          01:35:11 10.2.2.2        Eth1/2
```
```text
LEAF-02(config)# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/101], 01:34:43, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/101], 01:35:23, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:21, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:23, ospf-UNDERLAY, intra
10.0.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:23, ospf-UNDERLAY, intra
10.1.100.3/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:23, ospf-UNDERLAY, intra
10.1.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:21, ospf-UNDERLAY, intra
10.1.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:23, ospf-UNDERLAY, intra
10.1.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.2, Eth1/1, [110/201], 01:34:43, ospf-UNDERLAY, intra
    *via 10.2.2.2, Eth1/2, [110/201], 01:35:23, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/200], 01:34:43, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/200], 01:34:43, ospf-UNDERLAY, intra
10.2.1.6/31, ubest/mbest: 1/0
    *via 10.2.1.2, Eth1/1, [110/200], 01:34:43, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/200], 01:35:23, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/200], 01:35:23, ospf-UNDERLAY, intra
10.2.2.6/31, ubest/mbest: 1/0
    *via 10.2.2.2, Eth1/2, [110/200], 01:35:23, ospf-UNDERLAY, intra
```
```text
LEAF-02(config)# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.102.1, local AS number 65102
BGP table version is 192, L2VPN EVPN config peers 2, capable peers 2
22 network entries and 40 paths using 6088 bytes of memory
BGP attribute entries [16/2752], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    1953    1905      192    0    0 00:51:02 12
10.0.2.1        4 65999    1965    1918      192    0    0 00:50:59 12
```
```text
LEAF-02(config)# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 192, Local Router ID is 10.0.102.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.0.102.1:32777    (L2VNI 10010)
*>l[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                        100      32768 i
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65104 i
*>l[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                        100      32768 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.102.1:32787    (L2VNI 10020)
*>l[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                        100      32768 i
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65104 i
*>l[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                        100      32768 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.103.1:32777
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i
*>e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
* e                   10.1.100.3                                     0 65999 65103 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.103.1:32787
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65103 i
*>e                   10.1.100.3                                     0 65999 65103 i

Route Distinguisher: 10.0.104.1:32777
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i

Route Distinguisher: 10.0.104.1:32787
* e[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i
* e[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                                     0 65999 65104 i
*>e                   10.1.100.3                                     0 65999 65104 i

```
```text
LEAF-02(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
*   10     5006.0000.1b08   dynamic  0         F      F    Po10
C   10     5008.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
C   10     500a.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
*   20     5006.0000.1b08   dynamic  0         F      F    Po10
C   20     5008.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
C   20     500a.0000.1b08   dynamic  0         F      F    nve1(10.1.100.3)
G    -     5005.0000.1b08   static   -         F      F    sup-eth1(R)
G    -     5005.0000.1b08   static   -         F      F    Lo0(R) (Lo0)
```
```text
LEAF-02(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.100.3                              Up    CP        01:37:05 n/a

```
```text
LEAF-02(config)# sh vpc brief
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
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20


```


## LEAF-03
```text
LEAF-03(config)# sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  10.0.103.1      protocol-up/link-up/admin-up
Lo1                  10.1.103.1      protocol-up/link-up/admin-up
Eth1/1               10.2.1.5        protocol-up/link-up/admin-up
Eth1/2               10.2.2.5        protocol-up/link-up/admin-up
```
```text
LEAF-03(config)# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          01:39:13 10.2.1.4        Eth1/1
 10.0.2.1          1 FULL/ -          01:39:54 10.2.2.4        Eth1/2
```
```text
LEAF-03(config)# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/101], 01:39:26, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/101], 01:40:08, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:26, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:01, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:23, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:02, ospf-UNDERLAY, intra
10.0.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:23, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:03, ospf-UNDERLAY, intra
10.1.100.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:26, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:02, ospf-UNDERLAY, intra
10.1.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:26, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:01, ospf-UNDERLAY, intra
10.1.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:23, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:02, ospf-UNDERLAY, intra
10.1.104.1/32, ubest/mbest: 2/0
    *via 10.2.1.4, Eth1/1, [110/201], 01:39:23, ospf-UNDERLAY, intra
    *via 10.2.2.4, Eth1/2, [110/201], 01:40:03, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/200], 01:39:26, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/200], 01:39:26, ospf-UNDERLAY, intra
10.2.1.6/31, ubest/mbest: 1/0
    *via 10.2.1.4, Eth1/1, [110/200], 01:39:26, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/200], 01:40:08, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/200], 01:40:08, ospf-UNDERLAY, intra
10.2.2.6/31, ubest/mbest: 1/0
    *via 10.2.2.4, Eth1/2, [110/200], 01:40:08, ospf-UNDERLAY, intra
```
```text
LEAF-03(config)# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.103.1, local AS number 65103
BGP table version is 175, L2VPN EVPN config peers 2, capable peers 2
18 network entries and 30 paths using 4872 bytes of memory
BGP attribute entries [16/2752], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    2042    1999      175    0    0 00:55:38 8
10.0.2.1        4 65999    2055    2012      175    0    0 00:55:38 8
```
```text

LEAF-03(config)# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 175, Local Router ID is 10.0.103.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.0.101.1:32777
* e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
*>e                   10.1.100.1                                     0 65999 65101 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i

Route Distinguisher: 10.0.101.1:32787
* e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
*>e                   10.1.100.1                                     0 65999 65101 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i

Route Distinguisher: 10.0.102.1:32777
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i

Route Distinguisher: 10.0.102.1:32787
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i

Route Distinguisher: 10.0.103.1:32777    (L2VNI 10010)
* e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
*>e                   10.1.100.1                                     0 65999 65102 i
*>l[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>l[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65102 i
*>l[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                        100      32768 i

Route Distinguisher: 10.0.103.1:32787    (L2VNI 10020)
* e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
*>e                   10.1.100.1                                     0 65999 65102 i
*>l[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>l[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65102 i
*>l[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                        100      32768 i

```
```text
LEAF-03(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     5006.0000.1b08   dynamic  0         F      F    nve1(10.1.100.1)
*   10     5008.0000.1b08   dynamic  0         F      F    Po10
+   10     500a.0000.1b08   dynamic  0         F      F    Po100
C   20     5006.0000.1b08   dynamic  0         F      F    nve1(10.1.100.1)
*   20     5008.0000.1b08   dynamic  0         F      F    Po10
+   20     500a.0000.1b08   dynamic  0         F      F    Po100
G    -     5004.0000.1b08   static   -         F      F    sup-eth1(R)
G    -     5004.0000.1b08   static   -         F      F    Lo0(R) (Lo0)
LEAF-03(config)#
```
```text
LEAF-03(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.100.1                              Up    CP        01:41:26 n/a
```
```text
LEAF-03(config)# sh vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : primary
Number of vPCs configured         : 2
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20



100   Po100         up     success     success               10,20

```



## LEAF-04
```text
LEAF-04(config)#               sh ip int brief

IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Lo0                  10.0.104.1      protocol-up/link-up/admin-up
Lo1                  10.1.104.1      protocol-up/link-up/admin-up
Eth1/1               10.2.1.7        protocol-up/link-up/admin-up
Eth1/2               10.2.2.7        protocol-up/link-up/admin-up
```
```text
LEAF-04(config)# sh ip ospf neighbors
 OSPF Process ID UNDERLAY VRF default
 Total number of neighbors: 2
 Neighbor ID     Pri State            Up Time  Address         Interface
 10.0.1.1          1 FULL/ -          01:42:19 10.2.1.6        Eth1/1
 10.0.2.1          1 FULL/ -          01:43:02 10.2.2.6        Eth1/2
```
```text
LEAF-04(config)# sh ip route ospf-UNDERLAY
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.1.1/32, ubest/mbest: 1/0
    *via 10.2.1.6, Eth1/1, [110/101], 01:42:43, ospf-UNDERLAY, intra
10.0.2.1/32, ubest/mbest: 1/0
    *via 10.2.2.6, Eth1/2, [110/101], 01:43:26, ospf-UNDERLAY, intra
10.0.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:21, ospf-UNDERLAY, intra
10.0.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:22, ospf-UNDERLAY, intra
10.0.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:23, ospf-UNDERLAY, intra
10.1.100.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:22, ospf-UNDERLAY, intra
10.1.101.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra

...skipping 1 line
10.1.102.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:22, ospf-UNDERLAY, intra
10.1.103.1/32, ubest/mbest: 2/0
    *via 10.2.1.6, Eth1/1, [110/201], 01:42:43, ospf-UNDERLAY, intra
    *via 10.2.2.6, Eth1/2, [110/201], 01:43:23, ospf-UNDERLAY, intra
10.2.1.0/31, ubest/mbest: 1/0
    *via 10.2.1.6, Eth1/1, [110/200], 01:42:43, ospf-UNDERLAY, intra
10.2.1.2/31, ubest/mbest: 1/0
    *via 10.2.1.6, Eth1/1, [110/200], 01:42:43, ospf-UNDERLAY, intra
10.2.1.4/31, ubest/mbest: 1/0
    *via 10.2.1.6, Eth1/1, [110/200], 01:42:43, ospf-UNDERLAY, intra
10.2.2.0/31, ubest/mbest: 1/0
    *via 10.2.2.6, Eth1/2, [110/200], 01:43:26, ospf-UNDERLAY, intra
10.2.2.2/31, ubest/mbest: 1/0
    *via 10.2.2.6, Eth1/2, [110/200], 01:43:26, ospf-UNDERLAY, intra
10.2.2.4/31, ubest/mbest: 1/0
    *via 10.2.2.6, Eth1/2, [110/200], 01:43:26, ospf-UNDERLAY, intra
```
```text
LEAF-04(config)# sh bgp l2vpn evpn summary
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 10.0.104.1, local AS number 65104
BGP table version is 181, L2VPN EVPN config peers 2, capable peers 2
18 network entries and 30 paths using 4872 bytes of memory
BGP attribute entries [16/2752], BGP AS path entries [2/20]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.0.1.1        4 65999    2104    2063      181    0    0 00:58:42 8
10.0.2.1        4 65999    2117    2076      181    0    0 00:58:40 8
```
```text
LEAF-04(config)# sh bgp l2vpn evpn
BGP routing table information for VRF default, address family L2VPN EVPN
BGP table version is 181, Local Router ID is 10.0.104.1
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-i
njected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup, 2 - b
est2

   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 10.0.101.1:32777
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i

Route Distinguisher: 10.0.101.1:32787
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65101 i
* e                   10.1.100.1                                     0 65999 65101 i

Route Distinguisher: 10.0.102.1:32777
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i

Route Distinguisher: 10.0.102.1:32787
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65102 i

Route Distinguisher: 10.0.104.1:32777    (L2VNI 10010)
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65101 i
*>l[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>l[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65101 i
*>l[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                        100      32768 i

Route Distinguisher: 10.0.104.1:32787    (L2VNI 10020)
*>e[2]:[0]:[0]:[48]:[5006.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65101 i
*>l[2]:[0]:[0]:[48]:[5008.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>l[2]:[0]:[0]:[48]:[500a.0000.1b08]:[0]:[0.0.0.0]/216
                      10.1.100.3                        100      32768 i
*>e[3]:[0]:[32]:[10.1.100.1]/88
                      10.1.100.1                                     0 65999 65102 i
* e                   10.1.100.1                                     0 65999 65101 i
*>l[3]:[0]:[32]:[10.1.100.3]/88
                      10.1.100.3                        100      32768 i
```
```text
LEAF-04(config)# sh mac address-table
Legend:
        * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
        age - seconds since last seen,+ - primary entry using vPC Peer-Link,
        (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
   VLAN     MAC Address      Type      age     Secure NTFY Ports
---------+-----------------+--------+---------+------+----+------------------
C   10     5006.0000.1b08   dynamic  0         F      F    nve1(10.1.100.1)
+   10     5008.0000.1b08   dynamic  0         F      F    Po10
*   10     500a.0000.1b08   dynamic  0         F      F    Po100
C   20     5006.0000.1b08   dynamic  0         F      F    nve1(10.1.100.1)
+   20     5008.0000.1b08   dynamic  0         F      F    Po10
*   20     500a.0000.1b08   dynamic  0         F      F    Po100
G    -     5007.0000.1b08   static   -         F      F    sup-eth1(R)
G    -     5007.0000.1b08   static   -         F      F    Lo0(R) (Lo0)
```
```text
LEAF-04(config)# sh nve peers
Interface Peer-IP                                 State LearnType Uptime   Route
r-Mac
--------- --------------------------------------  ----- --------- -------- -----
------------
nve1      10.1.100.1                              Up    CP        00:59:28 n/a
```
```text
LEAF-04(config)# sh vpc brief
Legend:
                (*) - local vPC is down, forwarding via vPC peer-link

vPC domain id                     : 1
Peer status                       : peer adjacency formed ok
vPC keep-alive status             : peer is alive
Configuration consistency status  : success
Per-vlan consistency status       : success
Type-2 consistency status         : success
vPC role                          : secondary
Number of vPCs configured         : 2
Peer Gateway                      : Enabled
Dual-active excluded VLANs        : -
Graceful Consistency Check        : Enabled
Auto-recovery status              : Disabled
Delay-restore status              : Timer is off.(timeout = 30s)
Delay-restore SVI status          : Timer is off.(timeout = 10s)
Operational Layer3 Peer-router    : Enabled
Virtual-peerlink mode             : Disabled

vPC Peer-link status
---------------------------------------------------------------------
id    Port   Status Active vlans
--    ----   ------ -------------------------------------------------
1     Po1    up     1,10,20


vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20



100   Po100         up     success     success               10,20
```

---


# Проверка доступности

## HV-01
```text
HV-01# sh ip int brief vrf all

Interface            IP Address      Interface Status
Vlan10               172.25.81.5     protocol-up/link-up/admin-up

IP Interface Status for VRF "VLAN-20"(4)
Interface            IP Address      Interface Status
Vlan20               172.25.82.5     protocol-up/link-up/admin-up

```
```text
HV-01# ping 172.25.81.6 vrf VlAN-20
PING 172.25.81.6 (172.25.81.6): 56 data bytes
64 bytes from 172.25.81.6: icmp_seq=0 ttl=253 time=13.833 ms
64 bytes from 172.25.81.6: icmp_seq=1 ttl=253 time=11.374 ms
64 bytes from 172.25.81.6: icmp_seq=2 ttl=253 time=14.761 ms

```
```text
HV-01# ping 172.25.82.6 vrf VlAN-20
PING 172.25.82.6 (172.25.82.6): 56 data bytes
64 bytes from 172.25.82.6: icmp_seq=0 ttl=254 time=9.844 ms
64 bytes from 172.25.82.6: icmp_seq=1 ttl=254 time=8.784 ms
64 bytes from 172.25.82.6: icmp_seq=2 ttl=254 time=8.744 ms

```
```text
HV-01# traceroute 172.25.81.6 vrf VlAN-20
traceroute to 172.25.81.6 (172.25.81.6), 30 hops max, 40 byte packets
 1  172.25.82.1 (172.25.82.1)  10.738 ms  9.337 ms  8.599 ms
 2  172.25.81.6 (172.25.81.6)  13.543 ms  12.395 ms  11.447 ms
```

```text
HV-01# ping 172.25.81.6 vrf VlAN-10
PING 172.25.81.6 (172.25.81.6): 56 data bytes
64 bytes from 172.25.81.6: icmp_seq=0 ttl=254 time=15.787 ms
64 bytes from 172.25.81.6: icmp_seq=1 ttl=254 time=13.938 ms
64 bytes from 172.25.81.6: icmp_seq=2 ttl=254 time=15.96 ms
```

```text
HV-01# ping 172.25.82.6 vrf VlAN-10
PING 172.25.82.6 (172.25.82.6): 56 data bytes
64 bytes from 172.25.82.6: icmp_seq=0 ttl=253 time=17.521 ms
64 bytes from 172.25.82.6: icmp_seq=1 ttl=253 time=16.053 ms
64 bytes from 172.25.82.6: icmp_seq=2 ttl=253 time=16.015 ms
```

```text
HV-01# traceroute 172.25.82.6 vrf VlAN-10
traceroute to 172.25.82.6 (172.25.82.6), 30 hops max, 40 byte packets
 1  172.25.81.1 (172.25.81.1)  12.545 ms  11.46 ms  11.219 ms
 2  172.25.82.6 (172.25.82.6)  15.528 ms  11.459 ms  14.122 ms
```

## При отключении одного из линка Po10:

```text
LEAF-01(config-if)# sh int eth 1/6 status

--------------------------------------------------------------------------------
Port          Name               Status    Vlan      Duplex  Speed   Type
--------------------------------------------------------------------------------
Eth1/6        HV-01              channelDo trunk     auto    auto    10g

LEAF-01(config-if)# sh vpc brief vpc 10

vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          down*  success     success               -
```

```text
LEAF-02(config)# sh int eth 1/6 status

--------------------------------------------------------------------------------
Port          Name               Status    Vlan      Duplex  Speed   Type
--------------------------------------------------------------------------------
Eth1/6        --                 connected trunk     full    1000    10g


LEAF-02(config)# sh vpc  brief vpc 10
vPC status
----------------------------------------------------------------------------
Id    Port          Status Consistency Reason                Active vlans
--    ------------  ------ ----------- ------                ---------------
10    Po10          up     success     success               10,20
```

```text
HV-01(config-if)# ping 172.25.82.6 vrf VlAN-20
PING 172.25.82.6 (172.25.82.6): 56 data bytes
64 bytes from 172.25.82.6: icmp_seq=0 ttl=254 time=16.694 ms
64 bytes from 172.25.82.6: icmp_seq=1 ttl=254 time=11.456 ms
64 bytes from 172.25.82.6: icmp_seq=2 ttl=254 time=12.304 ms
```


```text
HV-01(config-if)# ping 172.25.81.6 vrf VlAN-20
PING 172.25.81.6 (172.25.81.6): 56 data bytes
64 bytes from 172.25.81.6: icmp_seq=0 ttl=253 time=14.402 ms
64 bytes from 172.25.81.6: icmp_seq=1 ttl=253 time=13.916 ms
64 bytes from 172.25.81.6: icmp_seq=2 ttl=253 time=14.875 ms
```

```text
HV-01(config-if)# traceroute 172.25.81.6 vrf VlAN-20
traceroute to 172.25.81.6 (172.25.81.6), 30 hops max, 40 byte packets
 1  172.25.82.1 (172.25.82.1)  11.794 ms  12.144 ms  16.85 ms
 2  172.25.81.6 (172.25.81.6)  14.134 ms  14.016 ms  15.297 ms
```
