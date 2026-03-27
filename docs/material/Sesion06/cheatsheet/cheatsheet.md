---
layout: default
---

# Cheatsheet: SQL — Parte 2
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-06](../../../sesiones/sesion-06.md)

---


# ⚡ 1. Diagnóstico Rápido

## Ver consultas lentas

```sql
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

## Analizar una query

```sql
EXPLAIN ANALYZE SELECT ...;
```

---

# 🔍 2. Lectura de EXPLAIN

## Scans

| Tipo             | Significado           | ¿Bueno? |
| ---------------- | --------------------- | ------- |
| Seq Scan         | Escanea toda la tabla | ❌       |
| Index Scan       | Usa índice            | ✅       |
| Bitmap Heap Scan | Combina índices       | ⚠️      |

---

## JOINs

| Tipo        | Uso             |
| ----------- | --------------- |
| Nested Loop | tablas pequeñas |
| Hash Join   | tablas grandes  |
| Merge Join  | datos ordenados |

---

## Métricas clave

* `actual time` → tiempo real
* `rows` → filas procesadas
* `loops` → repeticiones
* `buffers` → uso de memoria/disco

---

# 🧱 3. Índices (Reglas Clave)

## Crear índice básico

```sql
CREATE INDEX idx_col ON tabla(columna);
```

---

## ¿Dónde indexar?

✔ `WHERE`
✔ `JOIN`
✔ `ORDER BY`

---

## Índice compuesto

```sql
CREATE INDEX idx_user_fecha ON pedidos(usuario_id, fecha);
```

📌 Regla: **orden importa (izq → der)**

---

## Índice parcial

```sql
CREATE INDEX idx_activos 
ON sesiones(usuario_id)
WHERE activa = true;
```

✔ Más rápido
✔ Más pequeño

---

## Índice funcional

```sql
CREATE INDEX idx_lower_email 
ON usuarios (LOWER(email));
```

---

# 🧩 4. JSONB Optimization

## Consulta

```sql
WHERE metadata->>'tipo' = 'error';
```

## Índice recomendado

```sql
CREATE INDEX idx_tipo 
ON tabla ((metadata->>'tipo'));
```

---

## Alternativa (GIN)

```sql
CREATE INDEX idx_json 
ON tabla USING gin (metadata);
```

✔ Flexible
❌ Menos preciso

---

# 🚀 5. Reglas de Oro

## ❌ Evitar funciones en columnas indexadas

```sql
WHERE EXTRACT(YEAR FROM fecha) = 2025;
```

## ✅ Mejor

```sql
WHERE fecha >= '2025-01-01'
AND fecha < '2026-01-01';
```

---

## ❌ SELECT *

```sql
SELECT * FROM usuarios;
```

## ✅ Mejor

```sql
SELECT id, nombre FROM usuarios;
```

---

## ❌ IN con grandes datasets

```sql
WHERE id IN (SELECT ...);
```

## ✅ Mejor

```sql
WHERE EXISTS (...);
```

---

# 📊 6. Selectividad

✔ Índices funcionan mejor si filtran **pocas filas**

| Caso         | Eficiencia |
| ------------ | ---------- |
| email único  | 🔥         |
| género (50%) | ❌          |

---

# ⚖️ 7. Trade-Off Índices

| Acción | Impacto    |
| ------ | ---------- |
| SELECT | 🔼 mejora  |
| INSERT | 🔽 empeora |
| UPDATE | 🔽 empeora |

📌 +10% ~ +30% costo por índice

---

# 🧪 8. Flujo de Optimización

1. Detectar query lenta
2. `EXPLAIN ANALYZE`
3. Identificar:

   * Seq Scan
   * JOIN costoso
4. Crear índice
5. Medir mejora

---

# 🧠 9. Patrones Comunes

## Filtro por usuario + fecha

```sql
WHERE usuario_id = ? AND fecha >= ?
```

👉 Índice:

```sql
(usuario_id, fecha)
```

---

## JOIN típico

```sql
ON pedidos.usuario_id = usuarios.id
```

👉 Índice:

```sql
pedidos(usuario_id)
```

---

## ORDER BY + filtro

```sql
WHERE usuario_id = 10
ORDER BY fecha DESC;
```

👉 Índice:

```sql
(usuario_id, fecha DESC)
```

---

# 📈 10. Señales de Problema

🚨 Query lenta → revisar:

* Seq Scan en tabla grande
* Falta de índices
* Uso de funciones
* JOIN sin índice
* Alto `rows`

---

# 🧩 11. Mini Checklist

✔ ¿Tiene índice el WHERE?
✔ ¿JOIN usa índice?
✔ ¿Evito funciones en columnas?
✔ ¿EXPLAIN muestra Index Scan?
✔ ¿Medí antes/después?

---

# 🏁 Frase Clave (entrevistas)

> “Optimizar SQL no es escribir menos código, es leer menos datos.”


