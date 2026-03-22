---
layout: default
---

# Cheatsheet: Normalización e integridad técnica
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-04](../../../sesiones/sesion-04.md)

---

## Formas normales (resumen)

| FN | Idea breve |
| :--- | :--- |
| 1FN | Sin grupos repetitivos; valores atómicos |
| 2FN | Sin dependencias parciales de PK compuesta |
| 3FN | No dependencias transitivas desde la PK |

---

## Integridad

| Restricción | Garantiza |
| :--- | :--- |
| `PRIMARY KEY` | Unicidad + no nulo |
| `FOREIGN KEY` | Existencia en tabla referenciada |
| `UNIQUE` | Unicidad (opcional nulo según motor) |
| `CHECK` | Regla booleana en fila |

---

## ACID

| Letra | Significado |
| :--- | :--- |
| A | Todo o nada (transacción) |
| C | Reglas de esquema |
| I | Aislamiento entre transacciones concurrentes |
| D | Persistencia tras commit |

---

## Teoría

* [teoria2-bbdd.md](../../Sesion02/teoria/teoria2-bbdd.md)

---

## Puntos críticos

* **Over-normalizar** puede multiplicar joins; **under-normalizar** puede bloquear actualizaciones coherentes.
* Elegir **nivel de aislamiento** según carga de lectura vs escritura.

> *“La integridad declarada en el esquema es más barata que corregir datos a mano.”*
