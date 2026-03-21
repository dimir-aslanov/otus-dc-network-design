# Домашнее задание
VxLAN. L2 VNI
## Цель:
Настроить Overlay на основе VxLAN EVPN для L2 связанности между клиентами.

---
# Схема
<img width="548" height="379" alt="image" src="https://github.com/user-attachments/assets/0c84e39c-92bd-496e-9609-542ff5b25246" />


---
# Конфигурация eBGP EVPN VxLAN
## SPINE-01
```text
configure terminal

feature nv overlay
nv overlay evpn
feature bgp

route-map EVPN_NH_UNCHANGED permit
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
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

neighbor 10.0.102.1
remote-as 65102
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

neighbor 10.0.103.1
remote-as 65103
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

```
## SPINE-02
```text
configure terminal

feature nv overlay
nv overlay evpn
feature bgp

route-map EVPN_NH_UNCHANGED permit
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
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

neighbor 10.0.102.1
remote-as 65102
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

neighbor 10.0.103.1
remote-as 65103
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map EVPN_NH_UNCHANGED out

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

```
---
# Что не получилось в лабе
### BFD ISIS - при включении сразу падает ISIS 

# Вывод CLI

```
## LEAF-01
```text
```
## LEAF-02
```text
```
## LEAF-03
```text
