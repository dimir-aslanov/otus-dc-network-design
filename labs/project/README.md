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



# Вывод CLI


## LEAF-01
```text
LEAF-01# sh ip route vrf OTUS-VRF1

0.0.0.0/0, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:49:18, bgp-65101, external, tag 65999, s
egid: 10999 tunnelid: 0xa016403 encap: VXLAN

172.25.81.0/24, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 1d02h, direct
172.25.81.1/32, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 1d02h, local
172.25.81.5/32, ubest/mbest: 1/0, attached
    *via 172.25.81.5, Vlan10, [190/0], 00:50:11, hmm
172.25.81.6/32, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:49:18, bgp-65101, external, tag 65999, s
egid: 10999 tunnelid: 0xa016403 encap: VXLAN

LEAF-01# sh ip route vrf OTUS-VRF2
0.0.0.0/0, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 01:00:24, bgp-65101, external, tag 65999, s
egid: 10888 tunnelid: 0xa016403 encap: VXLAN

172.25.82.0/24, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 1d03h, direct
172.25.82.1/32, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 1d03h, local
172.25.82.5/32, ubest/mbest: 1/0, attached
    *via 172.25.82.5, Vlan20, [190/0], 01:01:17, hmm
172.25.82.6/32, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 01:00:24, bgp-65101, external, tag 65999, s
egid: 10888 tunnelid: 0xa016403 encap: VXLAN


LEAF-01# sh bgp l2vpn evpn route-type 5 vrf OTUS-VRF1
Route Distinguisher: 10.0.101.1:3    (L3VNI 10999)
BGP routing table entry for [5]:[0]:[0]:[0]:[0.0.0.0]/224, version 3049
Paths: (2 available, best #2)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
n HW
Multipath: eBGP

  Path type: external, path is valid, not best reason: newer EBGP path, no label
ed nexthop
             Imported from 10.0.104.1:3:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65104 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10999
      Extcommunity: RT:65101:10999 ENCAP:8 Router MAC:5007.0000.1b08

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 10.0.103.1:3:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65103 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10999
      Extcommunity: RT:65101:10999 ENCAP:8 Router MAC:5004.0000.1b08

  Path-id 1 not advertised to any peer

LEAF-01# sh bgp l2vpn evpn route-type 5 vrf OTUS-VRF2
Route Distinguisher: 10.0.101.1:4    (L3VNI 10888)
BGP routing table entry for [5]:[0]:[0]:[0]:[0.0.0.0]/224, version 3048
Paths: (2 available, best #2)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
n HW
Multipath: eBGP

  Path type: external, path is valid, not best reason: newer EBGP path, no label
ed nexthop
             Imported from 10.0.104.1:4:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65104 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10888
      Extcommunity: RT:65101:10888 ENCAP:8 Router MAC:5007.0000.1b08

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 10.0.103.1:4:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65103 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10888
      Extcommunity: RT:65101:10888 ENCAP:8 Router MAC:5004.0000.1b08

  Path-id 1 not advertised to any peer


```

## LEAF-02
```text
LEAF-02(config-if)# sh ip route vrf OTUS-VRF1
0.0.0.0/0, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:56:52, bgp-65102, external, tag 65999, s
egid: 10999 tunnelid: 0xa016403 encap: VXLAN

172.25.81.0/24, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 01:01:03, direct
172.25.81.1/32, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 01:01:03, local
172.25.81.5/32, ubest/mbest: 1/0, attached
    *via 172.25.81.5, Vlan10, [190/0], 00:57:13, hmm (no-redist)
172.25.81.6/32, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:57:05, bgp-65102, external, tag 65999, s
egid: 10999 tunnelid: 0xa016403 encap: VXLAN


LEAF-02(config-if)# sh ip route vrf OTUS-VRF2
0.0.0.0/0, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:58:35, bgp-65102, external, tag 65999, s
egid: 10888 tunnelid: 0xa016403 encap: VXLAN

172.25.82.0/24, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 01:02:46, direct
172.25.82.1/32, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 01:02:46, local
172.25.82.5/32, ubest/mbest: 1/0, attached
    *via 172.25.82.5, Vlan20, [190/0], 00:58:56, hmm (no-redist)
172.25.82.6/32, ubest/mbest: 1/0
    *via 10.1.100.3%default, [20/0], 00:58:48, bgp-65102, external, tag 65999, s
egid: 10888 tunnelid: 0xa016403 encap: VXLAN


LEAF-02(config-if)# sh bgp l2vpn evpn route-type 5 vrf OTUS-VRF1
Route Distinguisher: 10.0.102.1:3    (L3VNI 10999)
BGP routing table entry for [5]:[0]:[0]:[0]:[0.0.0.0]/224, version 626
Paths: (2 available, best #1)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
n HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 10.0.104.1:3:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65104 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10999
      Extcommunity: RT:65102:10999 ENCAP:8 Router MAC:5007.0000.1b08

  Path type: external, path is valid, not best reason: Neighbor Address, no labe
led nexthop
             Imported from 10.0.103.1:3:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65103 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10999
      Extcommunity: RT:65102:10999 ENCAP:8 Router MAC:5004.0000.1b08

  Path-id 1 not advertised to any peer

LEAF-02(config-if)# sh bgp l2vpn evpn route-type 5 vrf OTUS-VRF2
Route Distinguisher: 10.0.102.1:4    (L3VNI 10888)
BGP routing table entry for [5]:[0]:[0]:[0]:[0.0.0.0]/224, version 627
Paths: (2 available, best #1)
Flags: (0x000002) (high32 00000000) on xmit-list, is not in l2rib/evpn, is not i
n HW
Multipath: eBGP

  Advertised path-id 1
  Path type: external, path is valid, is best path, no labeled nexthop
             Imported from 10.0.104.1:4:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65104 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10888
      Extcommunity: RT:65102:10888 ENCAP:8 Router MAC:5007.0000.1b08

  Path type: external, path is valid, not best reason: Neighbor Address, no labe
led nexthop
             Imported from 10.0.103.1:4:[5]:[0]:[0]:[0]:[0.0.0.0]/224
  Gateway IP: 0.0.0.0
  AS-Path: 65999 65103 , path sourced external to AS
    10.1.100.3 (metric 201) from 10.0.2.1 (10.0.2.1)
      Origin IGP, MED not set, localpref 100, weight 0
      Received label 10888
      Extcommunity: RT:65102:10888 ENCAP:8 Router MAC:5004.0000.1b08

  Path-id 1 not advertised to any peer


```

## LEAF-03
```text
LEAF-03(config-if)# sh ip route vrf OTUS-VRF1
0.0.0.0/0, ubest/mbest: 1/0
    *via 172.25.99.2, [1/0], 01:24:04, static
172.25.81.0/24, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 22:25:58, direct
172.25.81.1/32, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 22:25:58, local
172.25.81.5/32, ubest/mbest: 1/0
    *via 10.1.100.1%default, [20/0], 01:00:50, bgp-65103, external, tag 65999, s
egid: 10999 tunnelid: 0xa016401 encap: VXLAN

172.25.81.6/32, ubest/mbest: 1/0, attached
    *via 172.25.81.6, Vlan10, [190/0], 01:01:16, hmm
172.25.99.0/30, ubest/mbest: 1/0, attached
    *via 172.25.99.1, Vlan777, [0/0], 01:24:51, direct
172.25.99.1/32, ubest/mbest: 1/0, attached
    *via 172.25.99.1, Vlan777, [0/0], 01:24:51, local

LEAF-03(config-if)# sh ip route vrf OTUS-VRF2
0.0.0.0/0, ubest/mbest: 1/0
    *via 172.25.98.2, [1/0], 01:24:39, static
172.25.82.0/24, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 22:26:33, direct
172.25.82.1/32, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 22:26:33, local
172.25.82.5/32, ubest/mbest: 1/0
    *via 10.1.100.1%default, [20/0], 01:01:25, bgp-65103, external, tag 65999, s
egid: 10888 tunnelid: 0xa016401 encap: VXLAN

172.25.82.6/32, ubest/mbest: 1/0, attached
    *via 172.25.82.6, Vlan20, [190/0], 01:01:51, hmm
172.25.98.0/30, ubest/mbest: 1/0, attached
    *via 172.25.98.1, Vlan666, [0/0], 01:25:26, direct
172.25.98.1/32, ubest/mbest: 1/0, attached
    *via 172.25.98.1, Vlan666, [0/0], 01:25:26, local


```

## LEAF-04
```text

LEAF-04(config-if)# sh ip route vrf OTUS-VRF1
0.0.0.0/0, ubest/mbest: 1/0
    *via 172.25.99.2, [1/0], 01:25:42, static
172.25.81.0/24, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 01:26:20, direct
172.25.81.1/32, ubest/mbest: 1/0, attached
    *via 172.25.81.1, Vlan10, [0/0], 01:26:20, local
172.25.81.5/32, ubest/mbest: 1/0
    *via 10.1.100.1%default, [20/0], 01:02:36, bgp-65104, external, tag 65999, s
egid: 10999 tunnelid: 0xa016401 encap: VXLAN

172.25.81.6/32, ubest/mbest: 1/0, attached
    *via 172.25.81.6, Vlan10, [190/0], 01:02:54, hmm
172.25.99.0/30, ubest/mbest: 1/0, attached
    *via 172.25.99.1, Vlan777, [0/0], 01:26:20, direct
172.25.99.1/32, ubest/mbest: 1/0, attached
    *via 172.25.99.1, Vlan777, [0/0], 01:26:20, local


LEAF-04(config-if)# sh ip route vrf OTUS-VRF2
0.0.0.0/0, ubest/mbest: 1/0
    *via 172.25.98.2, [1/0], 01:25:10, static
172.25.82.0/24, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 01:25:48, direct
172.25.82.1/32, ubest/mbest: 1/0, attached
    *via 172.25.82.1, Vlan20, [0/0], 01:25:48, local
172.25.82.5/32, ubest/mbest: 1/0
    *via 10.1.100.1%default, [20/0], 01:02:04, bgp-65104, external, tag 65999, s
egid: 10888 tunnelid: 0xa016401 encap: VXLAN

172.25.82.6/32, ubest/mbest: 1/0, attached
    *via 172.25.82.6, Vlan20, [190/0], 01:02:22, hmm
172.25.98.0/30, ubest/mbest: 1/0, attached
    *via 172.25.98.1, Vlan666, [0/0], 01:25:48, direct
172.25.98.1/32, ubest/mbest: 1/0, attached
    *via 172.25.98.1, Vlan666, [0/0], 01:25:48, local


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


HV-01# ping 172.25.82.6 vrf VLAN-10
PING 172.25.82.6 (172.25.82.6): 56 data bytes
64 bytes from 172.25.82.6: icmp_seq=0 ttl=250 time=17.724 ms
64 bytes from 172.25.82.6: icmp_seq=1 ttl=250 time=16.429 ms
64 bytes from 172.25.82.6: icmp_seq=2 ttl=250 time=24.292 ms

HV-01# traceroute 172.25.82.6 vrf VLAN-10
traceroute to 172.25.82.6 (172.25.82.6), 30 hops max, 40 byte packets
 1  172.25.81.1 (172.25.81.1)  3.401 ms  3.122 ms  2.236 ms
 2  172.25.81.1 (172.25.81.1)  9.843 ms  8.446 ms  7.82 ms
 3  172.25.99.2 (172.25.99.2)  12.769 ms  14.524 ms  12.855 ms
 4  172.25.98.1 (172.25.98.1)  23.518 ms  16.106 ms  16.05 ms
 5  172.25.82.6 (172.25.82.6)  18.068 ms  18.701 ms  18.479 ms

HV-01# ping 172.25.81.6 vrf VLAN-20
PING 172.25.81.6 (172.25.81.6): 56 data bytes
64 bytes from 172.25.81.6: icmp_seq=0 ttl=250 time=21.9 ms
64 bytes from 172.25.81.6: icmp_seq=1 ttl=250 time=19.947 ms
64 bytes from 172.25.81.6: icmp_seq=2 ttl=250 time=16.453 ms

HV-01# traceroute 172.25.81.6 vrf VLAN-20
traceroute to 172.25.81.6 (172.25.81.6), 30 hops max, 40 byte packets
 1  172.25.82.1 (172.25.82.1)  3.228 ms  2.63 ms  2.649 ms
 2  172.25.82.1 (172.25.82.1)  9.017 ms  10.639 ms  9.995 ms
 3  172.25.98.2 (172.25.98.2)  11.613 ms  10.288 ms  10.692 ms
 4  172.25.99.1 (172.25.99.1)  11.869 ms  13.05 ms  11.953 ms
 5  172.25.81.6 (172.25.81.6)  15.833 ms  17.102 ms  15.556 ms

```

## HV-02
```text
HV-02# show ip int brief vrf all

IP Interface Status for VRF "VLAN-10"(3)
Interface            IP Address      Interface Status
Vlan10               172.25.81.6     protocol-up/link-up/admin-up

IP Interface Status for VRF "VLAN-20"(4)
Interface            IP Address      Interface Status
Vlan20               172.25.82.6     protocol-up/link-up/admin-up

HV-02# ping 172.25.82.5 vrf VLAN-10
PING 172.25.82.5 (172.25.82.5): 56 data bytes
64 bytes from 172.25.82.5: icmp_seq=0 ttl=250 time=15.628 ms
64 bytes from 172.25.82.5: icmp_seq=1 ttl=250 time=16.526 ms
64 bytes from 172.25.82.5: icmp_seq=2 ttl=250 time=14.527 ms

HV-02# traceroute 172.25.82.5 vrf VLAN-10
traceroute to 172.25.82.5 (172.25.82.5), 30 hops max, 40 byte packets
 1  172.25.81.1 (172.25.81.1)  3.1 ms  3.02 ms  1.923 ms
 2  172.25.99.2 (172.25.99.2)  5.641 ms  4.55 ms  4.13 ms
 3  172.25.98.1 (172.25.98.1)  5.541 ms  5.826 ms  5.456 ms
 4  172.25.82.1 (172.25.82.1)  14.587 ms  16.142 ms  12.959 ms
 5  172.25.82.5 (172.25.82.5)  13.343 ms  15.131 ms  15.518 ms

HV-02# ping 172.25.81.5 vrf VLAN-20
PING 172.25.81.5 (172.25.81.5): 56 data bytes
64 bytes from 172.25.81.5: icmp_seq=0 ttl=250 time=22.762 ms
64 bytes from 172.25.81.5: icmp_seq=1 ttl=250 time=21.834 ms
64 bytes from 172.25.81.5: icmp_seq=2 ttl=250 time=17.55 ms

HV-02# traceroute 172.25.81.5 vrf VLAN-20
traceroute to 172.25.81.5 (172.25.81.5), 30 hops max, 40 byte packets
 1  172.25.82.1 (172.25.82.1)  4.676 ms  4.792 ms  2.298 ms
 2  172.25.98.2 (172.25.98.2)  4.852 ms  5.224 ms  9.029 ms
 3  172.25.99.1 (172.25.99.1)  8.039 ms  8.579 ms  7.024 ms
 4  172.25.81.1 (172.25.81.1)  12.792 ms  16.08 ms  13.564 ms
 5  172.25.81.5 (172.25.81.5)  15.109 ms  19.671 ms  14.963 ms
```

