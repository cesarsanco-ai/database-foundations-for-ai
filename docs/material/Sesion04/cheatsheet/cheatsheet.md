---
layout: default
---

# Cheatsheet: SQL - Parte 1
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-04](../../../sesiones/sesion-04.md)

---
## 🧩 Sublenguajes SQL

| Tipo    | Comandos                       | Uso           |
| ------- | ------------------------------ | ------------- |
| **DDL** | CREATE, ALTER, DROP            | Estructura    |
| **DML** | SELECT, INSERT, UPDATE, DELETE | Datos         |
| **DCL** | GRANT, REVOKE                  | Permisos      |
| **TCL** | COMMIT, ROLLBACK               | Transacciones |

---

## ⚙️ DML (día a día)

### SELECT

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna
LIMIT n;
```

---

### INSERT

```sql
INSERT INTO tabla (col1, col2)
VALUES (v1, v2);
```

---

### UPDATE

```sql
UPDATE tabla
SET columna = valor
WHERE condicion;
```

⚠️ Sin `WHERE` → actualizas todo

---

### DELETE

```sql
DELETE FROM tabla WHERE condicion;
```

---

### UPSERT (PostgreSQL)

```sql
INSERT ... ON CONFLICT DO UPDATE;
```

---

## 🔗 JOINs (CLAVE)

### Tipos

| JOIN      | Qué hace               |
| --------- | ---------------------- |
| **INNER** | Solo coincidencias     |
| **LEFT**  | Todo izquierda + match |
| **RIGHT** | Todo derecha           |
| **FULL**  | Todo ambos             |
| **CROSS** | Producto cartesiano    |

---

### Ejemplo

```sql
SELECT c.nombre, p.fecha
FROM clientes c
JOIN pedidos p ON c.id = p.cliente_id;
```

---

### 🧠 Regla mental

* 1:N → FK
* N:M → tabla intermedia + JOIN

---

## ⚡ Algoritmos de JOIN

| Tipo            | Cuándo                   |
| --------------- | ------------------------ |
| **Nested Loop** | tablas pequeñas / índice |
| **Hash Join**   | tablas grandes           |
| **Merge Join**  | datos ordenados          |

---

## 🔍 Orden REAL de ejecución

```sql
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

⚠️ Por eso:

* ❌ usar alias en WHERE
* ✔ usar alias en ORDER BY

---

## 🧠 Subconsultas

### Tipos

* **Escalar** → 1 valor
* **Múltiples filas** → IN, EXISTS
* **Correlacionada** → depende de la externa

---

### EXISTS vs IN

```sql
-- Mejor (rápido)
WHERE EXISTS (...)

-- Peor si hay muchos datos
WHERE IN (...)
```

---

## 🧱 CTE (WITH)

### Básico

```sql
WITH temp AS (
  SELECT ...
)
SELECT * FROM temp;
```

---

### Ventajas

* Más legible
* Reutilizable
* Permite recursión

---

### Recursivo

```sql
WITH RECURSIVE t AS (
  SELECT ...
  UNION ALL
  SELECT ...
)
```

👉 usado en jerarquías (organigramas)

---

## 🚀 Optimización SQL

### Reglas clave

✔ Usa índices
✔ Evita `SELECT *`
✔ Usa `EXISTS` en grandes datos
✔ Evita funciones en columnas indexadas
✔ Usa `UNION ALL` en vez de `UNION`

---

### ❌ Malo

```sql
WHERE EXTRACT(YEAR FROM fecha) = 2025
```

### ✔ Bueno

```sql
WHERE fecha >= '2025-01-01'
AND fecha < '2026-01-01'
```

---

## 🔍 EXPLAIN

```sql
EXPLAIN ANALYZE SELECT ...
```

Te dice:

* Plan de ejecución
* Coste
* Tiempo real
* Uso de índices

---

## ⚙️ Qué pasa internamente

1. Parsing → valida SQL
2. Optimización → mejor plan
3. Ejecución → usa CPU/RAM/Disco
4. Resultado → retorna datos

---

## 💻 Hardware importa

* **CPU** → cálculos
* **RAM (buffer)** → cache
* **SSD** → acceso rápido
* **Red** → envío datos

👉 Si está en RAM → 🔥 rápido
👉 Si va a disco → 🐢 lento

---

## ⚖️ Buenas prácticas

* Indexa columnas de JOIN
* Usa LEFT JOIN si necesitas “todos”
* Usa COALESCE para NULL → 0
* Escribe SQL legible

---

## 🧠 Patrones comunes

### Total por cliente

```sql
SELECT c.nombre, SUM(...)
FROM clientes c
LEFT JOIN pedidos p ...
GROUP BY c.nombre;
```

---

### Comparar con promedio

```sql
WHERE salario > (
  SELECT AVG(salario) ...
)
```

---

## 🔥 Tips de entrevista

* Explica JOINs + ejemplos
* Diferencia INNER vs LEFT
* Orden ejecución SQL
* Cuándo usar índices
* EXISTS vs IN
* Leer EXPLAIN

---

## ⚡ Regla de oro final

👉 **Menos datos + menos JOINs + buenos índices = SQL rápido**

---
