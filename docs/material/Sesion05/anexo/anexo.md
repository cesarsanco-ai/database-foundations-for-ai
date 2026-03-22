---
layout: default
---

# Fundamento: DML, JOINs y lógica de conjuntos en SQL
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-05](../../../sesiones/sesion-05.md)

*(Alineado con la teoría de la Semana 5: [teoria5-bbdd.md](../teoria/teoria5-bbdd.md).)*

---

## 1. Sublenguajes SQL

**DDL** (esquema), **DML** (datos: SELECT/INSERT/UPDATE/DELETE), **DCL** (permisos). Esta sesión centra el **DML** y combinación de tablas.

---

## 2. JOINs

`INNER`, `LEFT`, `RIGHT`, `FULL` — definir bien **condición de unión** y nulos en claves foráneas.

---

## 3. Subconsultas y CTEs

Subconsultas correlacionadas; **`WITH` (CTE)** para legibilidad y planes repetibles en motores modernos.

---

## 4. Conjuntos

`UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT` — semántica de duplicados y orden.

---

## 5. Optimización (introducción)

Planes de ejecución, estadísticas, índices adecuados; evitar **funciones sobre columnas indexadas** en WHERE.

---

## 6. Enlaces directos

* **Teoría completa:** [teoria5-bbdd.md](../teoria/teoria5-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
