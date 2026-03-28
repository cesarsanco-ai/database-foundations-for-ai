# 📘 Laboratorio 1: Análisis de Datos

## 🎯 Objetivo
Aplicar técnicas avanzadas de DML (agregaciones, funciones de ventana, limpieza de datos, subconsultas, CTEs) al caso del asistente virtual Aletza, preparando datos para análisis y consumo por modelos de IA.

---

## 📋 Requisitos Previos
- Tener ejecutado el Laboratorio 1 con todas las tablas creadas y datos insertados
- Conectado a Supabase con el SQL Editor abierto

---

## 🔍 Paso 1: Agregaciones y Agrupamientos (GROUP BY)

### 1.1 Funciones de Agregación Básicas

```sql
-- Contar total de usuarios
SELECT COUNT(*) AS total_usuarios FROM usuarios;

-- Contar usuarios con país registrado
SELECT COUNT(pais) AS usuarios_con_pais FROM usuarios;

-- Contar países distintos
SELECT COUNT(DISTINCT pais) AS paises_distintos FROM usuarios WHERE pais IS NOT NULL;

-- Suma total de muestras de voz
SELECT SUM(muestras_tomadas) AS total_muestras FROM perfiles_voz;

-- Promedio de tiempo de ejecución de módulos
SELECT AVG(tiempo_ejecucion_ms) AS tiempo_promedio_ms FROM logs_ejecucion;

-- Máximo y mínimo tiempo de ejecución
SELECT 
    MAX(tiempo_ejecucion_ms) AS tiempo_maximo,
    MIN(tiempo_ejecucion_ms) AS tiempo_minimo
FROM logs_ejecucion;
```

### 1.2 GROUP BY Básico

```sql
-- Contar usuarios por país
SELECT 
    pais, 
    COUNT(*) AS cantidad_usuarios
FROM usuarios
WHERE pais IS NOT NULL
GROUP BY pais
ORDER BY cantidad_usuarios DESC;

-- Contar mensajes por modalidad
SELECT 
    modalidad,
    COUNT(*) AS cantidad_mensajes
FROM mensajes
GROUP BY modalidad
ORDER BY cantidad_mensajes DESC;

-- Sesiones por canal
SELECT 
    canal,
    COUNT(*) AS total_sesiones
FROM sesiones
GROUP BY canal
ORDER BY total_sesiones DESC;

-- Tasa de éxito por módulo
SELECT 
    modulo,
    COUNT(*) AS total_ejecuciones,
    SUM(CASE WHEN exito THEN 1 ELSE 0 END) AS exitosas,
    ROUND(100.0 * SUM(CASE WHEN exito THEN 1 ELSE 0 END) / COUNT(*), 2) AS tasa_exito
FROM logs_ejecucion
GROUP BY modulo
ORDER BY tasa_exito DESC;
```

### 1.3 GROUP BY con Múltiples Columnas

```sql
-- Mensajes por usuario y modalidad
SELECT 
    u.username,
    m.modalidad,
    COUNT(*) AS cantidad
FROM mensajes m
INNER JOIN usuarios u ON m.remitente_id = u.id
GROUP BY u.username, m.modalidad
ORDER BY u.username, cantidad DESC;

-- Sesiones por usuario y canal
SELECT 
    u.username,
    s.canal,
    COUNT(*) AS sesiones
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id
GROUP BY u.username, s.canal
ORDER BY u.username, sesiones DESC;
```

### 1.4 HAVING (Filtrar Grupos)

```sql
-- Usuarios con más de 1 sesión
SELECT 
    u.username,
    COUNT(s.id) AS total_sesiones
FROM usuarios u
LEFT JOIN sesiones s ON u.id = s.usuario_id
GROUP BY u.id, u.username
HAVING COUNT(s.id) > 1
ORDER BY total_sesiones DESC;

-- Módulos con tasa de éxito menor a 95%
SELECT 
    modulo,
    COUNT(*) AS total,
    ROUND(100.0 * SUM(CASE WHEN exito THEN 1 ELSE 0 END) / COUNT(*), 2) AS tasa_exito
FROM logs_ejecucion
GROUP BY modulo
HAVING ROUND(100.0 * SUM(CASE WHEN exito THEN 1 ELSE 0 END) / COUNT(*), 2) < 95;

-- Países con más de 2 usuarios
SELECT 
    pais,
    COUNT(*) AS cantidad
FROM usuarios
WHERE pais IS NOT NULL
GROUP BY pais
HAVING COUNT(*) > 2;
```

### 1.5 ROLLUP, CUBE y GROUPING SETS

```sql
-- ROLLUP: Subtotales por país y ciudad
SELECT 
    pais,
    ciudad,
    COUNT(*) AS cantidad
FROM usuarios
WHERE pais IS NOT NULL
GROUP BY ROLLUP (pais, ciudad)
ORDER BY pais NULLS LAST, ciudad NULLS LAST;

-- CUBE: Todas las combinaciones
SELECT 
    modalidad,
    EXTRACT(YEAR FROM enviado_en) AS año,
    COUNT(*) AS cantidad
FROM mensajes
GROUP BY CUBE (modalidad, EXTRACT(YEAR FROM enviado_en))
ORDER BY modalidad, año;

-- GROUPING SETS: Combinaciones específicas
SELECT 
    modulo,
    EXTRACT(YEAR FROM ejecutado_en) AS año,
    COUNT(*) AS total,
    ROUND(AVG(tiempo_ejecucion_ms), 2) AS tiempo_promedio
FROM logs_ejecucion
GROUP BY GROUPING SETS (
    (modulo),
    (EXTRACT(YEAR FROM ejecutado_en)),
    (modulo, EXTRACT(YEAR FROM ejecutado_en)),
    ()
)
ORDER BY modulo NULLS LAST, año NULLS LAST;
```

---

## 📊 Paso 2: Funciones de Ventana (Window Functions)

### 2.1 Funciones de Ranking

```sql
-- Ranking de usuarios por cantidad de mensajes
SELECT 
    u.username,
    COUNT(m.id) AS mensajes,
    ROW_NUMBER() OVER (ORDER BY COUNT(m.id) DESC) AS row_num,
    RANK() OVER (ORDER BY COUNT(m.id) DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY COUNT(m.id) DESC) AS dense_rank,
    NTILE(3) OVER (ORDER BY COUNT(m.id) DESC) AS tercil
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username
ORDER BY mensajes DESC;

-- Top 3 módulos por tiempo de ejecución (por partición de éxito)
SELECT 
    modulo,
    tiempo_ejecucion_ms,
    exito,
    ROW_NUMBER() OVER (PARTITION BY exito ORDER BY tiempo_ejecucion_ms DESC) AS ranking
FROM logs_ejecucion
WHERE tiempo_ejecucion_ms IS NOT NULL
ORDER BY exito DESC, ranking;
```

### 2.2 Funciones de Valor (LAG, LEAD)

```sql
-- Comparar mensajes consecutivos de un usuario
SELECT 
    u.username,
    m.contenido,
    m.enviado_en,
    LAG(m.contenido, 1) OVER (PARTITION BY m.remitente_id ORDER BY m.enviado_en) AS mensaje_anterior,
    LEAD(m.contenido, 1) OVER (PARTITION BY m.remitente_id ORDER BY m.enviado_en) AS mensaje_siguiente,
    EXTRACT(EPOCH FROM (m.enviado_en - LAG(m.enviado_en, 1) OVER (PARTITION BY m.remitente_id ORDER BY m.enviado_en))) / 60 AS minutos_diferencia
FROM mensajes m
INNER JOIN usuarios u ON m.remitente_id = u.id
ORDER BY u.username, m.enviado_en;

-- Tiempo entre sesiones de un usuario
SELECT 
    u.username,
    s.iniciada_en,
    LAG(s.iniciada_en, 1) OVER (PARTITION BY s.usuario_id ORDER BY s.iniciada_en) AS sesion_anterior,
    EXTRACT(EPOCH FROM (s.iniciada_en - LAG(s.iniciada_en, 1) OVER (PARTITION BY s.usuario_id ORDER BY s.iniciada_en))) / 3600 AS horas_diferencia
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id
ORDER BY u.username, s.iniciada_en;
```

### 2.3 Funciones de Agregación como Ventana

```sql
-- Acumulado de mensajes por día
SELECT 
    DATE(enviado_en) AS fecha,
    COUNT(*) AS mensajes_dia,
    SUM(COUNT(*)) OVER (ORDER BY DATE(enviado_en)) AS acumulado_mensajes
FROM mensajes
GROUP BY DATE(enviado_en)
ORDER BY fecha;

-- Media móvil de 3 días de mensajes
SELECT 
    DATE(enviado_en) AS fecha,
    COUNT(*) AS mensajes_dia,
    AVG(COUNT(*)) OVER (ORDER BY DATE(enviado_en) ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS media_movil_3dias
FROM mensajes
GROUP BY DATE(enviado_en)
ORDER BY fecha;

-- Porcentaje del total de mensajes por usuario
SELECT 
    u.username,
    COUNT(m.id) AS mensajes,
    ROUND(100.0 * COUNT(m.id) / SUM(COUNT(m.id)) OVER (), 2) AS porcentaje_total
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username
ORDER BY mensajes DESC;
```




