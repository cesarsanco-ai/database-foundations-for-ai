---
layout: default
---

# Fundamento: agregaciones, ventanas, limpieza y ETL con SQL
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-06](../../../sesiones/sesion-06.md)

*(Alineado con la teoría de la Semana 6: [teoria6-bbdd.md](../teoria/teoria6-bbdd.md).)*

---

## 1. Agregaciones y `GROUP BY`

Funciones `COUNT, SUM, AVG, MIN, MAX` con agrupaciones; **HAVING** filtra grupos (no filas sueltas).

---

## 2. Funciones de ventana

`OVER (PARTITION BY ... ORDER BY ...)` — rankings, medias móviles, **lags** sin auto-joins pesados.

---

## 3. Limpieza de datos

Tratamiento de nulos, duplicados, fechas y texto (`TRIM`, regex según motor).

---

## 4. Tablas temporales y pivotes

Útiles en pipelines analíticos; cuidado con **volatilidad** y alcance en sesión.

---

## 5. ETL desde SQL

Preparar datasets para ML: muestreo, features agregadas, ventanas temporales coherentes.

---

## 6. Enlaces directos

* **Teoría completa:** [teoria6-bbdd.md](../teoria/teoria6-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
