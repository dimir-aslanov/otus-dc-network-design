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

#### Для Loopback
| Диапазон | Роль |
|---------|------|
| 1–99 | Spine |
| 101–199 | Leaf | 
| 200–220 | Border / Gateway |


## Правила адресации

- Loopback интерфейсы — `/32`
- P2P линковка — `/31`
- Spine — только underlay
- Leaf — underlay + overlay (VTEP)

## IP-plan




