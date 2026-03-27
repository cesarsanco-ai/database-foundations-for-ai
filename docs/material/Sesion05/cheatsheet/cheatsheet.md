---
layout: default
---

# Cheatsheet: SQL — Parte 2
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-05](../../../sesiones/sesion-05.md)

---


## 📊 Agregaciones (GROUP BY)

### Funciones básicas

* `COUNT(*)` → filas
* `SUM()` → suma
* `AVG()` → promedio
* `MIN() / MAX()` → extremos

---

### GROUP BY

```sql
SELECT depto, COUNT(*), AVG(salario)
FROM empleados
GROUP BY depto;
```

---

### HAVING (filtro de grupos)

```sql
HAVING AVG(salario) > 4000
```

---

## 🚀 Agregaciones avanzadas

| Función           | Qué hace            |
| ----------------- | ------------------- |
| **ROLLUP**        | subtotales + total  |
| **CUBE**          | todas combinaciones |
| **GROUPING SETS** | control total       |

---

## 🪟 Funciones de Ventana

👉 No agrupan → mantienen filas

```sql
funcion() OVER (...)
```

---

### PARTES CLAVE

* `PARTITION BY` → agrupa lógico
* `ORDER BY` → orden dentro grupo
* `ROWS` → ventana (rango)

---

### Ranking

| Función        | Diferencia  |
| -------------- | ----------- |
| `ROW_NUMBER()` | sin empates |
| `RANK()`       | con huecos  |
| `DENSE_RANK()` | sin huecos  |
| `NTILE(n)`     | divide en n |

---

### Ejemplo

```sql
SELECT nombre,
ROW_NUMBER() OVER (ORDER BY salario DESC)
FROM empleados;
```

---

### LAG / LEAD (🔥 muy importante)

```sql
SELECT fecha, monto,
LAG(monto) OVER (ORDER BY fecha)
FROM ventas;
```

👉 comparar filas (series temporales)

---

### Acumulados / medias móviles

```sql
SUM(monto) OVER (ORDER BY fecha)
```

```sql
AVG(monto) OVER (
  ORDER BY fecha
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

---

## 🧼 Limpieza de datos

### Nulos

* `COALESCE()` → reemplaza
* `NULLIF()` → convierte

---

### Texto

* `UPPER()`, `LOWER()`, `INITCAP()`
* `TRIM()` → limpia espacios
* `REPLACE()`
* `SUBSTRING()`

---

### Regex (🔥 entrevistas)

```sql
SUBSTRING(texto FROM '[0-9]{5}')
```

---

## 📅 Fechas

* `CURRENT_DATE`
* `EXTRACT(YEAR FROM fecha)`
* `DATE_TRUNC('month', fecha)`

---

### Ejemplo

```sql
WHERE fecha >= '2025-01-01'
```

---

## 🔄 Conversión

* `CAST()`
* `TO_DATE()`
* `TO_CHAR()`

---

## 🧠 CASE (condicional)

```sql
CASE
 WHEN salario < 2000 THEN 'bajo'
 WHEN salario < 4000 THEN 'medio'
 ELSE 'alto'
END
```

---

## 🔁 Pivot (filas → columnas)

```sql
SUM(CASE WHEN mes=1 THEN monto END)
```

👉 o:

```sql
SUM(monto) FILTER (WHERE mes=1)
```

---

## 🧱 ETL en SQL

### Tablas temporales

```sql
CREATE TEMP TABLE t AS SELECT ...
```

---

### CTE (WITH)

```sql
WITH t AS (...)
SELECT * FROM t;
```

---

## 🧠 Manejo de duplicados

```sql
ROW_NUMBER() OVER (PARTITION BY email)
```

👉 eliminar donde `rn > 1`

---

## 🔗 Operaciones de conjuntos

* `UNION` → sin duplicados
* `UNION ALL` → más rápido
* `INTERSECT` → intersección
* `EXCEPT` → diferencia

---

## 📊 Patrones clave

### Top N por grupo

```sql
ROW_NUMBER() OVER (PARTITION BY depto ORDER BY salario DESC)
```

---

### Media móvil

```sql
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
```

---

### Comparar con promedio

```sql
WHERE valor > (SELECT AVG(...))
```

---

## ⚡ Buenas prácticas

* Usa ventanas en vez de subqueries cuando puedas
* Usa `FILTER` para agregaciones limpias
* Evita duplicados con ROW_NUMBER
* Limpia datos antes de analizar
* Usa CTE para legibilidad

---

## 🔥 Tips de entrevista

* Diferencia GROUP BY vs WINDOW
* Explicar LAG/LEAD
* Saber hacer ranking
* Pivot con CASE
* Media móvil
* Limpieza con COALESCE

---

## ⚡ Regla final

👉 **Agregaciones resumen → Ventanas analizan → ETL transforma**
