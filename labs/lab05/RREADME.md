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

nv overlay evpn

route-map NH_UNCHANGED
permit 10
set ip next-hop unchanged
!
router bgp 65999
router-id 4.4.4.4
timers bgp 3 9
reconnect-interval 10
log-neighbor-changes
address-family l2vpn evpn
maximum-paths 10
retain route-target all
neighbor 1.1.1.1
remote-as 65001
update-source loopback0
ebgp-multihop 5
address-family l2vpn evpn
send-community
send-community extended
rewrite-evpn-rt-asn
route-map NH_UNCHANGED

```
## SPINE-02
```text

```
## LEAF-01
```text

```
## LEAF-02
```text

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
