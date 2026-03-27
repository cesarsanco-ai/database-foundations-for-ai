# Laboratorio 6: Optimización de Consultas con Índices – Métricas de Rendimiento en PostgreSQL

## Aplicado al Caso Aletza

---

### Objetivo

Aprender a identificar consultas lentas, crear índices adecuados y analizar el rendimiento de las operaciones de búsqueda utilizando herramientas de PostgreSQL (`EXPLAIN ANALYZE`, `pg_stat_statements`). Al finalizar, el estudiante será capaz de:

- Interpretar planes de ejecución.
- Elegir el tipo de índice apropiado según el patrón de consultas.
- Medir el impacto de los índices en la latencia y el uso de recursos.
- Aplicar técnicas avanzadas como índices parciales, compuestos y sobre JSONB.

---

## 1. Introducción Teórica

### 1.1 ¿Por qué optimizar consultas?

Las bases de datos almacenan datos que crecen con el tiempo. Una consulta que tarda milisegundos con miles de registros puede convertirse en minutos con millones. La optimización busca reducir el tiempo de respuesta, minimizar el uso de CPU y E/S, y mantener la escalabilidad.

### 1.2 Índices en PostgreSQL

- **B-Tree (por defecto):** Para igualdad y rangos (`=`, `<`, `>`, `BETWEEN`, `LIKE` con patrón fijo al inicio). Es la opción más común.
- **Hash:** Solo igualdad, útil en columnas con muchos valores distintos (pero no soporta rangos).
- **GiST, GIN:** Para tipos complejos como JSONB, arrays, texto completo, geometría.
- **BRIN:** Para tablas muy grandes con datos ordenados físicamente (ej. series temporales).
- **Índices parciales:** Solo indexan un subconjunto de filas (ej. `WHERE activo = true`).
- **Índices compuestos:** Sobre varias columnas; el orden importa (columna más selectiva primero).
- **Índices funcionales:** Sobre expresiones (ej. `LOWER(email)` para búsquedas case-insensitive).

### 1.3 Analizando planes de ejecución con `EXPLAIN ANALYZE`

- **`EXPLAIN`** muestra el plan estimado (sin ejecutar).
- **`EXPLAIN ANALYZE`** ejecuta la consulta y muestra tiempos reales, número de filas, etc.
- Elementos clave en el plan:
  - **Seq Scan**: Escaneo secuencial (lectura de todas las filas). Costo O(n).
  - **Index Scan**: Uso de un índice para localizar filas. Costo O(log n) + acceso a filas.
  - **Bitmap Heap Scan**: Combina varios índices mediante mapas de bits, eficiente cuando se usan múltiples condiciones.
  - **Nested Loop, Hash Join, Merge Join**: Formas de combinar tablas.
  - **Costos**: números como `cost=0.42..8.44 rows=100 width=...`. El primer número es el costo de inicio, el segundo el costo total estimado. Unidades arbitrarias (relativas).
- **Métricas útiles**: `actual time`, `rows`, `loops`, `buffers` (lecturas de caché/disco).

### 1.4 Estadísticas y monitoreo

- `pg_stat_statements`: extensión que registra estadísticas de todas las consultas (tiempo de ejecución, llamadas, etc.). Útil para identificar las consultas más costosas.
- `ANALYZE`: actualiza estadísticas que el optimizador usa para estimar costos.

---

## 2. Preparación del Entorno

Utilizaremos el esquema de Aletza con datos de prueba. Si no lo tienes aún, ejecuta los scripts de creación e inserción de los laboratorios anteriores. Asegúrate de tener al menos:

- Tablas: `usuario`, `perfil_voz`, `sesion`, `mensaje`, `log_ejecucion`.
- Datos: suficiente para notar diferencias (mínimo 1000 mensajes, 100 sesiones, 10 usuarios). Para este laboratorio, puedes generar datos masivos con `generate_series`.

### 2.1 Generar datos de prueba (opcional)

```sql
-- Insertar 100 usuarios
INSERT INTO usuario (username, email)
SELECT 
    'user' || i,
    'user' || i || '@example.com'
FROM generate_series(1, 100) AS i;

-- Insertar 10,000 sesiones (distribuidas aleatoriamente entre usuarios)
INSERT INTO sesion (usuario_id, token, canal, iniciada_en, ultima_actividad, expira_en)
SELECT 
    (random() * 99 + 1)::int,
    md5(random()::text),
    (ARRAY['telegram', 'web', 'app'])[floor(random() * 3 + 1)],
    NOW() - (random() * interval '30 days'),
    NOW() - (random() * interval '30 days'),
    NOW() + (random() * interval '1 hour')
FROM generate_series(1, 10000) AS i;

-- Insertar 100,000 mensajes (asociados a sesiones existentes)
INSERT INTO mensaje (sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en)
SELECT 
    (random() * 9999 + 1)::int,
    (random() * 99 + 1)::int,
    'Texto de ejemplo ' || i,
    (ARRAY['texto', 'audio', 'imagen', 'video'])[floor(random() * 4 + 1)],
    jsonb_build_object('intencion', (ARRAY['soporte', 'saludo', 'consulta'])[floor(random() * 3 + 1)],
                       'sentimiento', (ARRAY['positivo', 'negativo', 'neutral'])[floor(random() * 3 + 1)]),
    NOW() - (random() * interval '30 days')
FROM generate_series(1, 100000) AS i;

-- Insertar logs para cada mensaje (aprox. 3 logs por mensaje)
INSERT INTO log_ejecucion (mensaje_id, modulo, tiempo_ejecucion_ms, exito, ejecutado_en)
SELECT 
    (random() * 99999 + 1)::int,
    (ARRAY['transcripcion', 'clasificacion', 'generacion', 'analisis_imagen'])[floor(random() * 4 + 1)],
    (random() * 500)::int,
    random() > 0.05,
    NOW() - (random() * interval '30 days')
FROM generate_series(1, 300000) AS i;
```

---

## 3. Actividades

### Actividad 1: Identificar consultas lentas sin índices

**Problema:** Se necesita obtener el total de mensajes enviados por cada usuario en los últimos 7 días, agrupados por día y canal de sesión. La consulta inicial es:

```sql
SELECT 
    u.username,
    DATE(m.enviado_en) AS dia,
    s.canal,
    COUNT(*) AS total_mensajes
FROM mensaje m
JOIN sesion s ON m.sesion_id = s.id_sesion
JOIN usuario u ON m.remitente_id = u.id_usuario
WHERE m.enviado_en >= NOW() - INTERVAL '7 days'
GROUP BY u.username, dia, s.canal;
```

**Tarea:** Ejecuta la consulta con `EXPLAIN ANALYZE` y anota:
- Tiempo total de ejecución.
- Tipo de escaneo en cada tabla (Seq Scan, Index Scan, etc.).
- ¿Hay algún índice que se esté utilizando?

**Preguntas:**
1. ¿Qué columnas podrían beneficiarse de un índice para acelerar esta consulta?
2. Propón al menos dos índices para mejorar el rendimiento y justifica por qué.

---

### Actividad 2: Creación de índices y comparación

**Tarea:**
- Crea los índices que propusiste en la Actividad 1.
- Vuelve a ejecutar la consulta con `EXPLAIN ANALYZE`.
- Compara los tiempos y el plan de ejecución con la versión sin índices.

**Registra:**
- Índices creados.
- Tiempo antes/después.
- Cambios en el plan (¿aparece Index Scan, Bitmap Index Scan?).

---

### Actividad 3: Índices parciales para consultas frecuentes

**Escenario:** En Aletza, una consulta recurrente es obtener las sesiones activas de un usuario (que no hayan expirado) para mostrar su estado. La consulta típica es:

```sql
SELECT * FROM sesion 
WHERE usuario_id = 5 AND expira_en > NOW()
ORDER BY ultima_actividad DESC;
```

Actualmente, esta consulta usa un índice simple en `usuario_id`, pero aún debe verificar la condición de expiración.

**Tarea:**
1. Crea un índice parcial que solo indexe las sesiones no expiradas (`expira_en > NOW()`).
2. Compara el plan de ejecución con el índice existente (si lo hay) y sin índice.
3. ¿Qué ventaja aporta el índice parcial?

---

### Actividad 4: Índices funcionales y JSONB

**Escenario:** La tabla `mensaje` tiene un campo JSONB `metadata_ia` que contiene la intención y sentimiento. Se realizan consultas como:

```sql
SELECT * FROM mensaje
WHERE metadata_ia->>'intencion' = 'soporte'
  AND metadata_ia->>'sentimiento' = 'negativo';
```

Esta consulta es lenta porque no hay índice sobre las claves del JSONB.

**Tarea:**
- Crea un índice GIN sobre `metadata_ia`.
- Crea también un índice funcional sobre `(metadata_ia->>'intencion')`.
- Ejecuta la consulta con `EXPLAIN ANALYZE` antes y después de cada índice.
- ¿Cuál de los dos índices es más efectivo? ¿Por qué?

---

### Actividad 5: Uso de `pg_stat_statements` para identificar consultas problemáticas

**Procedimiento:**
1. Activa la extensión `pg_stat_statements` en tu base de datos (en Supabase ya está habilitada, pero puedes ejecutar `CREATE EXTENSION IF NOT EXISTS pg_stat_statements;`).
2. Reinicia las estadísticas con `SELECT pg_stat_statements_reset();`.
3. Ejecuta varias veces las consultas anteriores (puedes usar un script).
4. Consulta la vista `pg_stat_statements` para ver qué consultas son las más costosas en términos de tiempo total y llamadas.

```sql
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 5;
```

**Preguntas:**
- ¿Qué consulta aparece como la más costosa?
- ¿Cómo se relaciona con los índices que creaste?
- ¿Qué otra información útil proporciona esta vista?

---

### Actividad 6: Evaluación de impacto en escritura

**Reflexión:** Los índices aceleran las lecturas pero ralentizan las escrituras. Simula una carga de inserciones masivas en `mensaje` y `log_ejecucion` con y sin los índices creados.

**Tarea:**
- Mide el tiempo de inserción de 10,000 mensajes antes de crear los índices.
- Luego crea todos los índices sugeridos en las actividades anteriores.
- Mide nuevamente el tiempo de inserción de otros 10,000 mensajes.
- Calcula el porcentaje de degradación.
- ¿Crees que vale la pena el trade-off para el sistema Aletza? Justifica.

---

## 4. Solucionario

### Actividad 1: Identificar consultas lentas sin índices

**Plan de ejecución esperado (sin índices):**

```
HashAggregate  (cost=... rows=...)
  ->  Hash Join  (cost=... rows=...)
        ->  Hash Join  (cost=... rows=...)
              ->  Seq Scan on mensaje m  (cost=... rows=...)
                    Filter: (enviado_en >= now() - '7 days'::interval)
              ->  Hash  (cost=... rows=...)
                    ->  Seq Scan on sesion s
        ->  Hash  (cost=... rows=...)
              ->  Seq Scan on usuario u
```

**Tiempo estimado:** Puede ser de varios segundos dependiendo del volumen de datos (con 100k mensajes, >1 segundo).

**Preguntas:**
1. Columnas que se benefician de índices:
   - `mensaje.enviado_en` (filtro por fecha)
   - `mensaje.sesion_id` (JOIN con sesion)
   - `mensaje.remitente_id` (JOIN con usuario)
   - `sesion.canal` (agrupación)
   - `usuario.username` (agrupación)
2. Propuesta de índices:
   - `CREATE INDEX idx_mensaje_enviado_en ON mensaje(enviado_en);`
   - `CREATE INDEX idx_mensaje_sesion_id ON mensaje(sesion_id);`
   - `CREATE INDEX idx_mensaje_remitente_id ON mensaje(remitente_id);`
   - `CREATE INDEX idx_sesion_canal ON sesion(canal);` (si la cardinalidad es baja, puede no ser tan útil)
   - `CREATE INDEX idx_usuario_username ON usuario(username);`

---

### Actividad 2: Creación de índices y comparación

**Índices recomendados (mínimo para mejorar la consulta):**
```sql
CREATE INDEX idx_mensaje_enviado_en ON mensaje(enviado_en);
CREATE INDEX idx_mensaje_sesion_id ON mensaje(sesion_id);
CREATE INDEX idx_mensaje_remitente_id ON mensaje(remitente_id);
```

**Plan después (con índices):**
- Puede usar un **Index Scan** en `mensaje` para filtrar por fecha si el índice está disponible.
- Luego un **Nested Loop** con índices en `sesion` y `usuario`.
- O un **Bitmap Heap Scan** combinando índices.

**Tiempo:** Puede reducirse drásticamente (ej. de 2 segundos a 200 ms).

**Observación:** El optimizador puede decidir usar secuencial si la tabla es pequeña; con datos grandes el índice marca la diferencia.

---

### Actividad 3: Índices parciales

**Índice parcial:**
```sql
CREATE INDEX idx_sesion_activa ON sesion(usuario_id, ultima_actividad DESC)
WHERE expira_en > NOW();
```

**Plan antes:** Index Scan con filtro adicional.

**Plan después:** Index Scan que ya excluye las expiradas, reduciendo filas a escanear.

**Ventaja:** Menor tamaño del índice, menor mantenimiento, y evita tener que evaluar la condición `expira_en > NOW()` en el índice (solo se indexan las activas).

---

### Actividad 4: Índices funcionales y JSONB

**Índice GIN:**
```sql
CREATE INDEX idx_mensaje_metadata_gin ON mensaje USING gin (metadata_ia);
```

**Índice funcional:**
```sql
CREATE INDEX idx_mensaje_intencion ON mensaje ((metadata_ia->>'intencion'));
```

**Plan con GIN:** Usa `Bitmap Index Scan` sobre el índice GIN para filtrar por `@>` o `->>` si se usa el operador adecuado. Para la consulta dada, el GIN no es tan efectivo porque el filtro es sobre claves específicas. PostgreSQL puede usar el índice GIN para acelerar búsquedas por existencia de claves, pero para igualdad de valor es mejor un índice funcional.

**Plan con índice funcional:** Index Scan directo sobre `metadata_ia->>'intencion'`. Luego se aplica filtro adicional de sentimiento (que puede ser secuencial si no hay índice compuesto). Para mejorar más, se puede crear un índice compuesto funcional sobre `(metadata_ia->>'intencion', metadata_ia->>'sentimiento')`.

**Recomendación:** Para consultas sobre campos JSONB específicos, los índices funcionales suelen ser más eficientes que GIN cuando las consultas son muy selectivas.

---

### Actividad 5: Uso de `pg_stat_statements`

**Resultado esperado:** Las consultas con filtros sin índices aparecerán con alto `total_time` y `mean_time`. Después de crear índices, sus métricas mejoran.

**Ejemplo de consulta:**
```sql
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
WHERE query LIKE '%mensaje%'
ORDER BY total_time DESC;
```

**Utilidad:** Identifica qué consultas consumen más recursos, permitiendo priorizar optimizaciones.

---

### Actividad 6: Evaluación de impacto en escritura

**Procedimiento:**
```sql
-- Medir inserción sin índices (o con pocos índices)
\timing on
INSERT INTO mensaje (sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en)
SELECT ... FROM generate_series(...);
\timing off

-- Luego crear índices adicionales y medir otra inserción
```

**Resultado típico:** Puede haber un incremento del 10-30% en el tiempo de inserción por cada índice adicional. Pero si las lecturas son predominantes (como en un sistema de consulta), el trade-off es favorable.

**Justificación para Aletza:** El sistema tiene una carga mixta: muchas lecturas (autenticación, consultas de usuario) y escrituras moderadas (mensajes, logs). Los índices son esenciales para la experiencia del usuario, por lo que vale la pena el costo en escritura.

---

## 5. Reflexión Final

En este laboratorio hemos aprendido a utilizar herramientas de análisis de rendimiento en PostgreSQL, creando índices estratégicos para acelerar consultas típicas del asistente Aletza. La práctica con `EXPLAIN ANALYZE`, índices parciales, funcionales y sobre JSONB proporciona una base sólida para optimizar cualquier sistema de bases de datos.

**Próximos pasos:** En la siguiente sesión profundizaremos en funciones, procedimientos almacenados y triggers para automatizar reglas de negocio, como la actualización de la última actividad en sesiones o el cierre automático de sesiones expiradas.

---