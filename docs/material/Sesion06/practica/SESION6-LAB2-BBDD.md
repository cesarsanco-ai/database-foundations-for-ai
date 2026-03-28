# 📘 Laboratorio 2: Optimización de Consultas

## 🗂️ Paso 3: Análisis de Consultas Actuales

### 3.1 Consultas Problemáticas Identificadas

```sql
-- 1. Búsqueda por país
EXPLAIN ANALYZE
SELECT username, email FROM usuarios WHERE pais = 'Perú';

-- 2. Búsqueda por rango de fechas en mensajes
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE enviado_en BETWEEN '2024-01-15' AND '2024-01-16';

-- 3. JOIN con múltiples condiciones
EXPLAIN ANALYZE
SELECT u.username, l.modulo, AVG(l.tiempo_ejecucion_ms) as tiempo_promedio
FROM usuarios u
JOIN mensajes m ON u.id = m.remitente_id
JOIN logs_ejecucion l ON m.id = l.mensaje_id
GROUP BY u.username, l.modulo;

-- 4. Búsqueda en JSONB
EXPLAIN ANALYZE
SELECT contenido, metadata_ia->>'intencion' as intencion
FROM mensajes
WHERE metadata_ia->>'sentimiento' = 'positivo';

-- 5. LIKE en texto
EXPLAIN ANALYZE
SELECT * FROM mensajes WHERE contenido LIKE '%clima%';
```

---

## 🔨 Paso 4: Creación de Índices Estratégicos

### 4.1 Índices Simples B-Tree

```sql
-- Índice para búsqueda por país en usuarios
CREATE INDEX idx_usuarios_pais ON usuarios(pais);

-- Índice para fecha en mensajes
CREATE INDEX idx_mensajes_enviado_en ON mensajes(enviado_en);

-- Índice para modalidad en mensajes
CREATE INDEX idx_mensajes_modalidad ON mensajes(modalidad);

-- Índice para remitente_id en mensajes
CREATE INDEX idx_mensajes_remitente_id ON mensajes(remitente_id);

-- Índice para usuario_id en sesiones
CREATE INDEX idx_sesiones_usuario_id ON sesiones(usuario_id);

-- Índice para mensaje_id en logs
CREATE INDEX idx_logs_mensaje_id ON logs_ejecucion(mensaje_id);

-- Índice para módulo en logs
CREATE INDEX idx_logs_modulo ON logs_ejecucion(modulo);

-- Verificar creación
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE tablename = 'usuarios' AND indexname LIKE 'idx_%';
```

### 4.2 Índices Compuestos

```sql
-- Índice compuesto para JOIN entre usuarios y mensajes
CREATE INDEX idx_mensajes_remitente_modalidad ON mensajes(remitente_id, modalidad);

-- Índice compuesto para sesiones por usuario y fecha
CREATE INDEX idx_sesiones_usuario_fecha ON sesiones(usuario_id, iniciada_en);

-- Índice compuesto para logs por módulo y éxito
CREATE INDEX idx_logs_modulo_exito ON logs_ejecucion(modulo, exito);

-- Índice compuesto para mensajes por fecha y modalidad
CREATE INDEX idx_mensajes_fecha_modalidad ON mensajes(enviado_en, modalidad);
```

### 4.3 Índices Parciales

```sql
-- Índice solo para perfiles completos
CREATE INDEX idx_perfiles_completos ON perfiles_voz(usuario_id) 
WHERE estado = 'completo';

-- Verificar índices parciales
SELECT indexname, indexdef 
FROM pg_indexes 
WHERE indexname LIKE 'idx_%' AND indexdef LIKE '%WHERE%';
```

### 4.4 Índices Funcionales

```sql
-- Índice para búsqueda de email en minúsculas
CREATE INDEX idx_usuarios_email_lower ON usuarios(LOWER(email));

-- Índice para búsqueda de contenido sin sensibilidad a mayúsculas
CREATE INDEX idx_mensajes_contenido_lower ON mensajes(LOWER(contenido));

-- Índice para extraer intención de JSONB
CREATE INDEX idx_mensajes_intencion ON mensajes((metadata_ia->>'intencion'));

-- Índice para extraer sentimiento de JSONB
CREATE INDEX idx_mensajes_sentimiento ON mensajes((metadata_ia->>'sentimiento'));

-- Prueba de índice funcional
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE LOWER(contenido) LIKE '%clima%';
```

### 4.5 Índices JSONB con GIN

```sql
-- Índice GIN para búsqueda en JSONB completo
CREATE INDEX idx_mensajes_metadata_gin ON mensajes USING gin(metadata_ia);

-- Verificar tamaño de índices
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS tamaño
FROM pg_indexes 
WHERE tablename IN ('usuarios', 'mensajes', 'sesiones', 'logs_ejecucion')
ORDER BY tamaño DESC;
```

---

## 📈 Paso 5: Medición de Rendimiento Antes/Después

### 5.1 Configurar Métricas de Prueba

```sql
-- Función para medir tiempo de ejecución
CREATE OR REPLACE FUNCTION medir_tiempo(consulta TEXT)
RETURNS TABLE(consulta_sql TEXT, tiempo_ms NUMERIC) AS $$
DECLARE
    inicio TIMESTAMP;
    fin TIMESTAMP;
BEGIN
    inicio := clock_timestamp();
    EXECUTE consulta;
    fin := clock_timestamp();
    
    RETURN QUERY SELECT consulta, EXTRACT(EPOCH FROM (fin - inicio)) * 1000;
END;
$$ LANGUAGE plpgsql;

-- Nota: Esta función es solo para referencia, usaremos EXPLAIN ANALYZE directamente
```

### 5.2 Comparativa Antes vs Después

```sql
-- ====== PRUEBA 1: Búsqueda por país ======
-- ANTES (sin índice)
EXPLAIN ANALYZE
SELECT username, email FROM usuarios WHERE pais = 'Perú';

-- DESPUÉS (con índice idx_usuarios_pais)
EXPLAIN ANALYZE
SELECT username, email FROM usuarios WHERE pais = 'Perú';
-- Esperado: Seq Scan → Index Scan

-- ====== PRUEBA 2: Rango de fechas ======
-- ANTES
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE enviado_en BETWEEN '2024-01-15' AND '2024-01-16';

-- DESPUÉS (con índice idx_mensajes_enviado_en)
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE enviado_en BETWEEN '2024-01-15' AND '2024-01-16';
-- Esperado: Seq Scan → Index Scan

-- ====== PRUEBA 3: JOIN con múltiples tablas ======
-- ANTES
EXPLAIN ANALYZE
SELECT u.username, l.modulo, AVG(l.tiempo_ejecucion_ms)
FROM usuarios u
JOIN mensajes m ON u.id = m.remitente_id
JOIN logs_ejecucion l ON m.id = l.mensaje_id
GROUP BY u.username, l.modulo;

-- DESPUÉS (con índices compuestos)
EXPLAIN ANALYZE
SELECT u.username, l.modulo, AVG(l.tiempo_ejecucion_ms)
FROM usuarios u
JOIN mensajes m ON u.id = m.remitente_id
JOIN logs_ejecucion l ON m.id = l.mensaje_id
GROUP BY u.username, l.modulo;
-- Esperado: Mejor uso de índices

-- ====== PRUEBA 4: JSONB ======
-- ANTES
EXPLAIN ANALYZE
SELECT contenido, metadata_ia->>'intencion'
FROM mensajes
WHERE metadata_ia->>'sentimiento' = 'positivo';

-- DESPUÉS (con índice funcional)
EXPLAIN ANALYZE
SELECT contenido, metadata_ia->>'intencion'
FROM mensajes
WHERE metadata_ia->>'sentimiento' = 'positivo';
-- Esperado: Seq Scan → Index Scan

-- ====== PRUEBA 5: LIKE en texto ======
-- ANTES
EXPLAIN ANALYZE
SELECT * FROM mensajes WHERE contenido LIKE '%clima%';

-- DESPUÉS (con índice funcional)
EXPLAIN ANALYZE
SELECT * FROM mensajes WHERE LOWER(contenido) LIKE '%clima%';
-- Esperado: Seq Scan → Index Scan (mejorado)
```

---

## 🔍 Paso 6: Uso de pg_stat_statements

### 6.1 Configurar y Ver Estadísticas

```sql
-- Verificar que pg_stat_statements está activo
SELECT * FROM pg_stat_statements LIMIT 1;

-- Reiniciar estadísticas para prueba limpia
SELECT pg_stat_statements_reset();

-- Ejecutar consultas de prueba varias veces para generar estadísticas
SELECT COUNT(*) FROM usuarios WHERE pais = 'Perú';
SELECT COUNT(*) FROM usuarios WHERE pais = 'Perú';
SELECT COUNT(*) FROM usuarios WHERE pais = 'Perú';

SELECT COUNT(*) FROM mensajes WHERE enviado_en > '2024-01-15';
SELECT COUNT(*) FROM mensajes WHERE enviado_en > '2024-01-15';

-- Consultar estadísticas
SELECT 
    query,
    calls,
    total_exec_time AS tiempo_total_ms,
    mean_exec_time AS tiempo_promedio_ms,
    rows,
    shared_blks_hit + shared_blks_read AS bloques_leidos
FROM pg_stat_statements
WHERE query NOT LIKE '%pg_stat_statements%'
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 6.2 Análisis de Métricas

```sql
-- Identificar consultas con mayor tiempo total
SELECT 
    LEFT(query, 100) AS query_resumida,
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    ROUND(mean_exec_time::numeric, 2) AS promedio_ms,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;

-- Identificar consultas más frecuentes
SELECT 
    LEFT(query, 100) AS query_resumida,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS promedio_ms,
    rows
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 5;

-- Identificar consultas con mayor promedio (lentas)
SELECT 
    LEFT(query, 100) AS query_resumida,
    calls,
    ROUND(mean_exec_time::numeric, 2) AS promedio_ms
FROM pg_stat_statements
WHERE calls > 5
ORDER BY mean_exec_time DESC
LIMIT 5;
```

---

## 🛠️ Paso 7: Optimización de Consultas Específicas

### 7.1 Optimizar Búsqueda en JSONB

```sql
-- Consulta original
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE metadata_ia->>'intencion' = 'clima';

-- Crear índice funcional específico
CREATE INDEX idx_mensajes_intencion_clima 
ON mensajes ((metadata_ia->>'intencion'));

-- Consulta optimizada
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE metadata_ia->>'intencion' = 'clima';
```

### 7.2 Optimizar LIKE con Búsqueda Parcial

```sql
-- Consulta original (lenta en tablas grandes)
EXPLAIN ANALYZE
SELECT * FROM usuarios WHERE username LIKE '%dev%';

-- Alternativa: usar índice de texto completo (si fuera necesario)
-- Para este caso, con pocos datos no es problema
```

### 7.3 Optimizar JOIN Complejo

```sql
-- Consulta original
EXPLAIN ANALYZE
SELECT 
    u.username,
    COUNT(DISTINCT s.id) AS sesiones,
    COUNT(m.id) AS mensajes,
    COUNT(l.id) AS ejecuciones
FROM usuarios u
LEFT JOIN sesiones s ON u.id = s.usuario_id
LEFT JOIN mensajes m ON s.id = m.sesion_id
LEFT JOIN logs_ejecucion l ON m.id = l.mensaje_id
GROUP BY u.id, u.username;

-- Versión optimizada usando subconsultas
EXPLAIN ANALYZE
WITH sesiones_count AS (
    SELECT usuario_id, COUNT(*) AS total_sesiones
    FROM sesiones
    GROUP BY usuario_id
),
mensajes_count AS (
    SELECT sesion_id, COUNT(*) AS total_mensajes
    FROM mensajes
    GROUP BY sesion_id
),
logs_count AS (
    SELECT mensaje_id, COUNT(*) AS total_logs
    FROM logs_ejecucion
    GROUP BY mensaje_id
)
SELECT 
    u.username,
    COALESCE(sc.total_sesiones, 0) AS sesiones,
    COALESCE(SUM(mc.total_mensajes), 0) AS mensajes,
    COALESCE(SUM(lc.total_logs), 0) AS ejecuciones
FROM usuarios u
LEFT JOIN sesiones_count sc ON u.id = sc.usuario_id
LEFT JOIN mensajes m ON u.id = m.remitente_id
LEFT JOIN mensajes_count mc ON m.id = mc.sesion_id
LEFT JOIN logs_count lc ON m.id = lc.mensaje_id
GROUP BY u.username, sc.total_sesiones;
```

---

## 📊 Paso 8: Análisis de Trade-offs

### 8.1 Medir Impacto en Escritura

```sql
-- Medir tiempo de inserción con índices
-- Crear tabla temporal para prueba
CREATE TEMP TABLE prueba_inserts (
    id SERIAL PRIMARY KEY,
    username TEXT,
    email TEXT,
    pais TEXT
);

-- Insertar 1000 registros (aproximado)
INSERT INTO prueba_inserts (username, email, pais)
SELECT 
    'user_' || generate_series,
    'user_' || generate_series || '@test.com',
    'País ' || (generate_series % 10)
FROM generate_series(1, 1000);

-- Agregar índices a tabla de prueba
CREATE INDEX idx_prueba_pais ON prueba_inserts(pais);
CREATE INDEX idx_prueba_username ON prueba_inserts(username);

-- Medir tiempo de inserción con índices
\timing on
INSERT INTO prueba_inserts (username, email, pais)
SELECT 
    'new_user_' || generate_series,
    'new_user_' || generate_series || '@test.com',
    'Nuevo País'
FROM generate_series(1, 100);
\timing off

-- Limpiar tabla de prueba
DROP TABLE prueba_inserts;
```

### 8.2 Evaluar Tamaño de Índices

```sql
-- Calcular tamaño total de tablas e índices
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS tamaño_total,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS tamaño_tabla,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS tamaño_indices
FROM pg_tables
WHERE schemaname = 'public'
AND tablename IN ('usuarios', 'mensajes', 'sesiones', 'logs_ejecucion', 'perfiles_voz')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Verificar índices por tabla
SELECT 
    tablename,
    COUNT(*) AS cantidad_indices,
    pg_size_pretty(SUM(pg_relation_size(indexname::regclass))) AS tamaño_indices
FROM pg_indexes
WHERE schemaname = 'public'
AND tablename IN ('usuarios', 'mensajes', 'sesiones', 'logs_ejecucion', 'perfiles_voz')
GROUP BY tablename
ORDER BY cantidad_indices DESC;
```


---

## 📚 Resumen de Técnicas de Optimización

| Técnica | Aplicación en Aletza | Impacto Esperado |
|---------|---------------------|------------------|
| **Índices B-Tree** | pais, fecha, usuario_id | Seq Scan → Index Scan |
| **Índices Compuestos** | (usuario_id, fecha) | JOINs más eficientes |
| **Índices Parciales** | usuarios activos, sesiones activas | Menor tamaño, más rápidos |
| **Índices Funcionales** | LOWER(email), JSONB keys | Búsquedas con funciones |
| **Índices GIN** | metadata JSONB completo | Búsquedas flexibles |
| **Reescritura de Consultas** | CTEs, subconsultas | Reducción de lecturas |
| **pg_stat_statements** | Monitoreo continuo | Identificación de problemas |

---

## 🎯 Conclusión

La optimización de consultas en PostgreSQL para el caso Aletza requiere:

1. **Medir** antes de optimizar (EXPLAIN ANALYZE)
2. **Identificar** cuellos de botella (Seq Scan, JOINs costosos)
3. **Aplicar** índices según patrones de acceso
4. **Evaluar** trade-offs entre lectura y escritura
5. **Monitorear** continuamente con pg_stat_statements

