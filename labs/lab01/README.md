# Домашнее задание

## Цель:

1) Собрать схему CLOS;  
2) Распределить адресное пространство.
   
----------------

# Целевая схема в EVE-NG

<img width="458" height="371" alt="image" src="https://github.com/user-attachments/assets/2f1e5977-3b34-4255-9a15-458d3e623c31" />

----------------

# Aдресное пространство

## Общий формат

`10.<Dn>.<Host>.<X>`

## Назначение октетов

### Dn — номер DC и тип сети  
Для **DC1 используется диапазон `0–10`**

| Dn | Назначение |
|----|------------|
| 0 | Loopback Underlay - Leaf and Spine |
| 1 | Loopback Overlay - Leaf | 
| 2 | P2P |
| 3 | Reserved |
| 4–7 | Services / VRF |
| 8–9 | Management |

### Host — идентификатор узла

#### Для Loopback | Management
| Диапазон | Роль |
|---------|------|
| 1–99 | Spine |
| 101–199 | Leaf | 
| 200–220 | Border / Gateway |

#### Для P2P - привязка к SPINE
| Диапазон | Номер SPINE|
|---------|------|
| 1–99 | № Spine |

## Правила адресации

- Loopback интерфейсы — `/32`
- P2P — `/31`
- Spine — только underlay
- Leaf — underlay + overlay (VTEP)

## IP-план

## SPINE-01

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-01 | Lo0 (loopback1)     | 10.0.1.1   | /32 |
| SPINE-01 | Lo1 (loopback2)     | 10.1.1.1   | /32 |
| SPINE-01 | Eth1/1 → LEAF-01    | 10.2.1.0   | /31 |
| SPINE-01 | Eth1/2 → LEAF-02    | 10.2.1.2   | /31 |
| SPINE-01 | Eth1/3 → LEAF-03    | 10.2.1.4   | /31 |

---

## SPINE-02

| Device   | Interface           | IP Address | Subnet Mask |
|----------|---------------------|------------|-------------|
| SPINE-02 | Lo0 (loopback1)     | 10.0.2.1   | /32 |
| SPINE-02 | Lo1 (loopback2)     | 10.1.2.1   | /32 |
| SPINE-02 | Eth1/1 → LEAF-01    | 10.2.2.0   | /31 |
| SPINE-02 | Eth1/2 → LEAF-02    | 10.2.2.2   | /31 |
| SPINE-02 | Eth1/3 → LEAF-03    | 10.2.2.4   | /31 |

---

## LEAF-01

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-01 | Lo0 (loopback1)     | 10.0.101.1 | /32 |
| LEAF-01 | Lo1 (loopback2)     | 10.1.101.1 | /32 |
| LEAF-01 | Eth1/1 → SPINE-01   | 10.2.1.1   | /31 |
| LEAF-01 | Eth1/2 → SPINE-02   | 10.2.2.1   | /31 |

---

## LEAF-02

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-02 | Lo0 (loopback1)     | 10.0.102.1 | /32 |
| LEAF-02 | Lo1 (loopback2)     | 10.1.102.1 | /32 |
| LEAF-02 | Eth1/1 → SPINE-01   | 10.2.1.3   | /31 |
| LEAF-02 | Eth1/2 → SPINE-02   | 10.2.2.3   | /31 |

---

## LEAF-03

| Device  | Interface           | IP Address | Subnet Mask |
|---------|---------------------|------------|-------------|
| LEAF-03 | Lo0 (loopback1)     | 10.0.103.1 | /32 |
| LEAF-03 | Lo1 (loopback2)     | 10.1.103.1 | /32 |
| LEAF-03 | Eth1/1 → SPINE-01   | 10.2.1.5   | /31 |
| LEAF-03 | Eth1/2 → SPINE-02   | 10.2.2.5   | /31 |





