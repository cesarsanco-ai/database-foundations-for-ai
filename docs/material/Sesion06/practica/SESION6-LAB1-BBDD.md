# 📘 Laboratorio 1: Optimización de Consultas

## 🎯 Objetivo

Desarrollar la capacidad de diagnosticar, medir y optimizar consultas SQL en Supabase (PostgreSQL), utilizando herramientas y técnicas avanzadas para el caso del asistente virtual Aletza.

Al finalizar, podrás:
- Interpretar planes de ejecución (`EXPLAIN ANALYZE`)
- Diseñar índices eficientes según patrones de acceso
- Medir rendimiento con métricas reales
- Optimizar consultas complejas (JOINs, filtros, JSONB)
- Evaluar trade-offs entre lectura y escritura

---

## 📋 Requisitos Previos
- Tener ejecutados los Laboratorios 1 y 2 con todas las tablas y datos
- Conectado a Supabase con el SQL Editor abierto
- Habilitar extensiones necesarias

---

## 🔧 Paso 1: Configuración Inicial en Supabase

### 1.1 Habilitar Extensiones Necesarias

```sql
-- Habilitar pg_stat_statements para métricas de rendimiento
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Verificar que la extensión está instalada
SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';

-- Habilitar auto_explain para log automático de consultas lentas
-- NOTA: En Supabase gratuito, algunas configuraciones pueden estar limitadas
-- ALTER SYSTEM SET auto_explain.log_min_duration = '100ms';
-- ALTER SYSTEM SET auto_explain.log_analyze = true;
-- SELECT pg_reload_conf();
```

### 1.2 Verificar Tablas y Datos Actuales

```sql
-- Verificar cantidad de registros en cada tabla
SELECT 
    'usuarios' AS tabla, COUNT(*) AS registros FROM usuarios
UNION ALL
SELECT 'perfiles_voz', COUNT(*) FROM perfiles_voz
UNION ALL
SELECT 'sesiones', COUNT(*) FROM sesiones
UNION ALL
SELECT 'mensajes', COUNT(*) FROM mensajes
UNION ALL
SELECT 'logs_ejecucion', COUNT(*) FROM logs_ejecucion;
```

---

## 📊 Paso 2: Introducción a EXPLAIN ANALYZE

### 2.1 Consulta Básica sin Índices

```sql
-- Consulta simple sin filtrar
EXPLAIN ANALYZE
SELECT * FROM mensajes;

-- Observar: Seq Scan (lectura secuencial de toda la tabla)
-- Tiempo: ~0.05-0.10 ms (con datos pequeños, no es problema)
```

### 2.2 Consulta con Filtro WHERE

```sql
-- Consulta con filtro por fecha
EXPLAIN ANALYZE
SELECT * FROM mensajes 
WHERE enviado_en > '2024-01-15';

-- Observar: Seq Scan (no hay índice en enviado_en)
-- Tiempo: aún rápido por pocos datos
```

### 2.3 Consulta con JOIN

```sql
-- Consulta con JOIN
EXPLAIN ANALYZE
SELECT u.username, m.contenido, m.enviado_en
FROM mensajes m
JOIN usuarios u ON m.remitente_id = u.id
WHERE m.modalidad = 'texto';

-- Observar: Hash Join o Nested Loop
-- El optimizador elige según estadísticas
```

### 2.4 Interpretación de Planes

```sql
-- Ejecutar consulta con análisis detallado
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT u.username, COUNT(m.id) AS total_mensajes
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username
ORDER BY total_mensajes DESC;
```

**Métricas clave a observar:**
- `actual time`: tiempo real de ejecución (inicio..fin)
- `rows`: número de filas procesadas
- `loops`: número de iteraciones
- `buffers`: lectura de datos (shared hit = caché, shared read = disco)
- `Planning Time`: tiempo de planificación
- `Execution Time`: tiempo de ejecución

### Definiciones de términos en el plan

* **Seq Scan (Sequential Scan)**
  Escaneo secuencial: el motor lee toda la tabla fila por fila. Se usa cuando no hay índice útil o la tabla es pequeña.

* **Hash**
  Construcción de una tabla hash en memoria para acelerar operaciones posteriores (usualmente en joins o agregaciones).

* **Hash Right Join**
  Un tipo de combinación (join) entre dos tablas donde se crea una tabla hash con una y se compara con la otra, preservando todas las filas de la tabla derecha.

* **HashAggregate**
  Operación que agrupa filas utilizando una estructura hash para calcular agregados (como SUM, COUNT, AVG) eficientemente.

* **Sort**
  Ordenamiento de las filas resultantes según algún criterio (por ejemplo, `ORDER BY`).

* **Cost**
  Estimación interna que hace el planificador sobre el “costo” de cada paso en términos de uso de CPU, disco y memoria.

* **Estimated Rows**
  Número de filas que el planificador espera procesar o devolver en cada paso.

* **Total Time**
  Tiempo total real que tomó la consulta para ejecutarse, medido en milisegundos.

