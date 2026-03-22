---
layout: default
---

# Cheatsheet: SQL — Parte 2
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-06](../../../sesiones/sesion-06.md)

---

## Ventana (plantilla)

```sql
SELECT
  fecha,
  importe,
  SUM(importe) OVER (PARTITION BY cliente_id ORDER BY fecha
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS acumulado
FROM pedido;
```

---

## GROUP BY vs WINDOW

| Necesidad | Herramienta |
| :--- | :--- |
| Una fila por grupo | `GROUP BY` |
| Conservar filas + métrica móvil | `OVER` |

---

## Limpieza (ideas)

* `COALESCE`, `NULLIF`  
* Deduplicar con `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`  

---

## Teoría

* [teoria6-bbdd.md](../teoria/teoria6-bbdd.md)

---

## Puntos críticos

* **Ventanas** mal particionadas = coste O(n²) en peor caso en planes ingenuos; revisar índices alineados con `PARTITION BY` / `ORDER BY`.
* Features para ML: **filtrar fugas temporales** (no usar futuro en train).

> *“El SQL analítico es el primer feature store de muchos equipos.”*
