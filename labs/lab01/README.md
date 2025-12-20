Домашнее задание
----------------

Проектирование адресного пространства

## Цель:

### Собрать схему CLOS;  
### Распределить адресное пространство.

## Spine1

| Device | Interface | IP Address | Subnet Mask |
|-------|-----------|------------|-------------|
| Spine1 | Lo0 (loopback1) | 10.0.1.1 | /32 |
| Spine1 | Lo1 (loopback2) | 10.1.1.1 | /32 |
| Spine1 | Eth1/1 → Leaf1 | 10.2.1.0 | /31 |
| Spine1 | Eth1/2 → Leaf2 | 10.2.1.2 | /31 |
| Spine1 | Eth1/3 → Leaf3 | 10.2.1.4 | /31 |

---

## Spine2

| Device | Interface | IP Address | Subnet Mask |
|-------|-----------|------------|-------------|
| Spine2 | Lo0 (loopback1) | 10.0.2.1 | /32 |
| Spine2 | Lo1 (loopback2) | 10.1.2.1 | /32 |
| Spine2 | Eth1/1 → Leaf1 | 10.2.2.0 | /31 |
| Spine2 | Eth1/2 → Leaf2 | 10.2.2.2 | /31 |
| Spine2 | Eth1/3 → Leaf3 | 10.2.2.4 | /31 |

---

## Leaf1

| Device | Interface | IP Address | Subnet Mask |
|-------|-----------|------------|-------------|
| Leaf1 | Lo0 (loopback1) | 10.0.1.2 | /32 |
| Leaf1 | Lo1 (loopback2) | 10.1.1.2 | /32 |
| Leaf1 | Eth1/1 → Spine1 | 10.2.1.1 | /31 |
| Leaf1 | Eth1/2 → Spine2 | 10.2.2.1 | /31 |

---

## Leaf2

| Device | Interface | IP Address | Subnet Mask |
|-------|-----------|------------|-------------|
| Leaf2 | Lo0 (loopback1) | 10.0.2.2 | /32 |
| Leaf2 | Lo1 (loopback2) | 10.1.2.2 | /32 |
| Leaf2 | Eth1/1 → Spine1 | 10.2.1.3 | /31 |
| Leaf2 | Eth1/2 → Spine2 | 10.2.2.3 | /31 |

---

## Leaf3

| Device | Interface | IP Address | Subnet Mask |
|-------|-----------|------------|-------------|
| Leaf3 | Lo0 (loopback1) | 10.0.3.2 | /32 |
| Leaf3 | Lo1 (loopback2) | 10.1.3.2 | /32 |
| Leaf3 | Eth1/1 → Spine1 | 10.2.1.5 | /31 |
| Leaf3 | Eth1/2 → Spine2 | 10.2.2.5 | /31 |

