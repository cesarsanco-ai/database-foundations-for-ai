---
layout: default
---

# Cheatsheet: SQL — Parte 1
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-05](../../../sesiones/sesion-05.md)

---

## JOINs

| Tipo | Conserva |
| :--- | :--- |
| `INNER` | Solo coincidencias |
| `LEFT` | Filas izquierda + match derecha |
| `FULL` | Ambos lados |

---

## Plantilla

```sql
SELECT a.id, b.nombre
FROM tabla_a a
LEFT JOIN tabla_b b ON b.a_id = a.id
WHERE a.activo = true;
```

---

## CTE

```sql
WITH ventas AS (
  SELECT cliente_id, SUM(importe) AS total
  FROM pedido
  GROUP BY cliente_id
)
SELECT * FROM ventas WHERE total > 1000;
```

---

## Teoría

* [teoria5-bbdd.md](../teoria/teoria5-bbdd.md)

---

## Puntos críticos

* `JOIN` sin `ON` / `WHERE` correcto ⇒ **producto cartesiano**.
* `UNION` elimina duplicados (coste); `UNION ALL` si ya son únicos.

> *“Escribe SQL legible: el optimizador ayuda, pero no adivina la intención.”*
