# Домашнее задание
Underlay. eBGP
## Цель:
Настроить eBGP для Underlay сети.

---
# Схема
<img width="449" height="371" alt="image" src="https://github.com/user-attachments/assets/9125a61a-93fb-4a3c-a830-00f7c96b8eab" />

---
# Конфигурация eBGP
## SPINE-01
```text
configure terminal

feature bfd
feature bgp
system jumbomtu 9216

interface loopback0
  description Router-ID
  ip address 10.0.1.1/32
  no shutdown

interface Ethernet1/1
  description to-LEAF-01_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.0/31
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/2
  description to-LEAF-02_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.2/31
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

interface Ethernet1/3
  description to-LEAF-03_Eth1/1
  no switchport
  mtu 9216
  ip address 10.2.1.4/31
  bfd interval 100 min_rx 100 multiplier 3
  no bfd echo
  no shutdown

router bgp 65000
  router-id 10.0.1.1
  log-neighbor-changes
  bestpath as-path multipath-relax
  reconnect-interval 3
  maxas-limit 50

  address-family ipv4 unicast
    maximum-paths 10
    bgp pic

  template peer LEAFS
    remote-as 65100-65200
    bfd
      interval 100 min_rx 100 multiplier 3
    timers 1 3
    address-family ipv4 unicast
      route-map RM_LEAFS_IN in
      maximum-prefix 100 80 restart 5

  bgp listen range 10.2.1.0/24 peer-group LEAFS
  bgp listen range 10.2.2.0/24 peer-group LEAFS


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
```
