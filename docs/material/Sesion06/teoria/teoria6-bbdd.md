
# Optimización de Consultas en SQL

## Enfoque en PostgreSQL: Índices, Métricas y Rendimiento

---

## 🎯 Objetivo

Desarrollar la capacidad de diagnosticar, medir y optimizar consultas SQL en entornos reales, utilizando herramientas y técnicas avanzadas de PostgreSQL.

Al finalizar, podrás:

* Interpretar planes de ejecución (`EXPLAIN ANALYZE`)
* Diseñar índices eficientes según patrones de acceso
* Medir rendimiento con métricas reales
* Optimizar consultas complejas (JOINs, filtros, JSONB)
* Evaluar trade-offs entre lectura y escritura

---

# 1. Fundamentos de Optimización

## 1.1 ¿Por qué optimizar consultas?

A medida que los datos crecen:

* Las consultas pasan de **milisegundos → segundos/minutos**
* Aumenta el uso de:

  * CPU
  * Memoria (RAM)
  * E/S de disco

**Objetivo de optimización:**

* Reducir latencia (tiempo de respuesta)
* Minimizar lecturas innecesarias
* Mejorar escalabilidad

---

## 1.2 Principio clave

> **“No optimices SQL, optimiza accesos a datos.”**

La mayor parte del costo proviene de:

* Leer demasiadas filas (**I/O bound**)
* No usar índices
* Mala selectividad

---

# 2. Índices en PostgreSQL (Clave del Rendimiento)

## 2.1 Tipos de Índices

### 🔹 B-Tree (por defecto)

* Igualdad y rangos:

  ```sql
  =, <, >, BETWEEN, ORDER BY
  ```
* También útil para:

  ```sql
  LIKE 'texto%'
  ```

👉 Es el **80% de los casos reales**

---

### 🔹 Hash

* Solo igualdad (`=`)
* Uso limitado en práctica moderna

---

### 🔹 GIN (Generalized Inverted Index)

Ideal para:

* JSONB
* Arrays
* Búsqueda de texto

```sql
CREATE INDEX idx_json ON tabla USING gin (columna_json);
```

---

### 🔹 GiST

* Geoespacial
* Tipos complejos

---

### 🔹 BRIN

* Tablas enormes
* Datos ordenados físicamente (logs, series temporales)

👉 Muy liviano, pero menos preciso

---

## 2.2 Tipos de diseño de índices

### 🔸 Índices simples

```sql
CREATE INDEX idx_fecha ON pedidos(fecha);
```

---

### 🔸 Índices compuestos

```sql
CREATE INDEX idx_user_fecha ON pedidos(usuario_id, fecha);
```

📌 Regla clave:

> El orden importa (izquierda → derecha)

---

### 🔸 Índices parciales

```sql
CREATE INDEX idx_activos ON sesiones(usuario_id)
WHERE activa = true;
```

✔ Menor tamaño
✔ Más rápidos
✔ Menor costo de mantenimiento

---

### 🔸 Índices funcionales

```sql
CREATE INDEX idx_lower_email ON usuarios (LOWER(email));
```

✔ Permiten usar índices en expresiones

---

### 🔸 Índices sobre JSONB

```sql
CREATE INDEX idx_json_intencion 
ON mensajes ((metadata->>'intencion'));
```

---

# 3. Planes de Ejecución (`EXPLAIN ANALYZE`)

## 3.1 ¿Qué es?

Permite ver **cómo PostgreSQL ejecuta una consulta**.

```sql
EXPLAIN ANALYZE SELECT * FROM pedidos;
```

---

## 3.2 Operaciones clave

### 🔴 Seq Scan (malo en tablas grandes)

```text
Seq Scan on tabla
```

* Lee toda la tabla
* Complejidad: O(n)

---

### 🟢 Index Scan

```text
Index Scan using idx_x
```

* Usa índice
* Mucho más eficiente

---

### 🟡 Bitmap Heap Scan

* Combina múltiples índices
* Útil para condiciones múltiples

---

## 3.3 JOINs internos

* **Nested Loop** → bueno para pocos datos
* **Hash Join** → bueno para grandes volúmenes
* **Merge Join** → cuando hay orden

---

## 3.4 Métricas importantes

Dentro de `EXPLAIN ANALYZE`:

* `actual time`
* `rows`
* `loops`
* `buffers`

Ejemplo:

```text
actual time=0.03..120.50 rows=1000 loops=1
```

---

# 4. Métricas de Rendimiento en PostgreSQL

## 4.1 `pg_stat_statements`

Extensión clave para producción.

```sql
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

Consulta:

```sql
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

---

## 4.2 Métricas importantes

* `calls` → frecuencia
* `total_time` → impacto total
* `mean_time` → latencia promedio
* `rows` → volumen retornado

---

## 4.3 Interpretación

| Métrica           | Qué indica           |
| ----------------- | -------------------- |
| Alto `mean_time`  | consulta lenta       |
| Alto `calls`      | consulta frecuente   |
| Alto `total_time` | mayor impacto global |

---

# 5. Estrategias de Optimización

## 5.1 Indexar columnas correctas

Indexar:

* Columnas en `WHERE`
* Columnas en `JOIN`
* Columnas en `ORDER BY`

---

## 5.2 Evitar funciones en columnas indexadas

❌ Malo:

```sql
WHERE EXTRACT(YEAR FROM fecha) = 2025;
```

✔ Bueno:

```sql
WHERE fecha >= '2025-01-01'
  AND fecha < '2026-01-01';
```

---

## 5.3 Selectividad

Un índice funciona mejor cuando:

* Filtra pocas filas

❌ Ejemplo malo:

```sql
WHERE genero = 'M'; -- 50% de la tabla
```

✔ Mejor:

```sql
WHERE email = 'x@mail.com';
```

---

## 5.4 Evitar `SELECT *`

* Más datos → más I/O
* Peor uso de índices

---

## 5.5 Uso de índices compuestos

Consulta:

```sql
WHERE usuario_id = 10 AND fecha > NOW() - INTERVAL '7 days';
```

Índice ideal:

```sql
(usuario_id, fecha)
```

---

# 6. JSONB y Optimización

## 6.1 Problema común

```sql
WHERE metadata->>'tipo' = 'error';
```

➡ Sin índice → lento

---

## 6.2 Soluciones

### Opción 1: GIN

```sql
CREATE INDEX idx_json ON tabla USING gin (metadata);
```

✔ Flexible
❌ Menos eficiente en igualdad específica

---

### Opción 2: Índice funcional (mejor)

```sql
CREATE INDEX idx_tipo ON tabla ((metadata->>'tipo'));
```

✔ Más rápido para igualdad

---

# 7. Trade-Off: Lectura vs Escritura

## 7.1 Problema

Cada índice:

* Mejora SELECT
* Empeora INSERT / UPDATE / DELETE

---

## 7.2 Impacto típico

* +10% a +30% en tiempo de escritura por índice

---

## 7.3 Regla práctica

| Sistema      | Estrategia        |
| ------------ | ----------------- |
| OLTP         | índices moderados |
| Analytics    | muchos índices    |
| Logs masivos | BRIN              |

---

# 8. Flujo Profesional de Optimización

## Paso 1: Detectar problema

* `pg_stat_statements`
* Logs lentos

---

## Paso 2: Analizar consulta

```sql
EXPLAIN ANALYZE
```

---

## Paso 3: Identificar cuello de botella

* Seq Scan
* JOIN costoso
* Filtro ineficiente

---

## Paso 4: Aplicar solución

* Crear índice
* Reescribir query
* Reducir datos

---

## Paso 5: Medir impacto

Antes vs después:

* Tiempo
* Filas
* Buffers

---

# 9. Ejemplo Integrado

## Consulta

```sql
SELECT *
FROM pedidos
WHERE usuario_id = 10
AND fecha >= NOW() - INTERVAL '7 days';
```

---

## Problema

* Seq Scan
* Alta latencia

---

## Solución

```sql
CREATE INDEX idx_user_fecha ON pedidos(usuario_id, fecha);
```

---

## Resultado esperado

* Index Scan
* Reducción drástica de tiempo

---

# 10. Buenas Prácticas

✔ Usar `EXPLAIN ANALYZE` siempre
✔ Medir antes y después
✔ Crear índices según consultas reales
✔ Evitar sobre-indexar
✔ Usar índices compuestos correctamente
✔ Revisar `pg_stat_statements` regularmente

---

# 11. Reflexión Final

La optimización no es solo técnica, es estratégica:

* No se trata de **más índices**, sino de **mejores índices**
* No se trata de consultas complejas, sino de **consultas eficientes**
* Lo importante es medir, no asumir

> **“Lo que no se mide, no se puede optimizar.”**


