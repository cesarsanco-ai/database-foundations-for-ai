
## LAB 2: Transformación de Datos

### 3.1 Manejo de Valores Nulos

```sql
-- Reemplazar NULLs en país
SELECT 
    username,
    COALESCE(pais, 'No especificado') AS pais_normalizado
FROM usuarios;

-- Reemplazar NULLs en ciudad
SELECT 
    username,
    COALESCE(ciudad, 'Sin ciudad') AS ciudad
FROM usuarios;

-- NULLIF: evitar división por cero
SELECT 
    modulo,
    tiempo_ejecucion_ms,
    NULLIF(tiempo_ejecucion_ms, 0) AS tiempo_seguro
FROM logs_ejecucion;

-- Combinar COALESCE con NULLIF
SELECT 
    modulo,
    COALESCE(NULLIF(tiempo_ejecucion_ms, 0), 1) AS tiempo_default
FROM logs_ejecucion;
```

### 3.2 Funciones de Texto

```sql
-- Normalizar emails a minúsculas
SELECT 
    username,
    LOWER(email) AS email_normalizado
FROM usuarios;

-- Extraer dominio del email
SELECT 
    username,
    email,
    SUBSTRING(email FROM '@(.*)$') AS dominio
FROM usuarios;

-- Longitud del username
SELECT 
    username,
    LENGTH(username) AS longitud_username
FROM usuarios
ORDER BY longitud_username DESC;

-- Concatenar nombre de usuario con email
SELECT 
    username || ' (' || email || ')' AS usuario_completo
FROM usuarios;

-- Reemplazar caracteres en contenido
SELECT 
    contenido,
    REPLACE(contenido, 'Aletza', 'ALETZA') AS contenido_modificado
FROM mensajes
WHERE contenido LIKE '%Aletza%';
```

### 3.3 Expresiones Regulares (PostgreSQL)

```sql
-- Extraer palabras que empiezan con mayúscula
SELECT 
    contenido,
    REGEXP_MATCHES(contenido, '[A-Z][a-z]+', 'g') AS palabras_mayuscula
FROM mensajes
WHERE contenido IS NOT NULL
LIMIT 5;

-- Validar formato de email
SELECT 
    username,
    email,
    email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' AS email_valido
FROM usuarios;

-- Reemplazar múltiples espacios por uno solo
SELECT 
    contenido,
    REGEXP_REPLACE(contenido, '\s+', ' ', 'g') AS contenido_limpio
FROM mensajes
WHERE contenido IS NOT NULL;
```

### 3.4 Funciones de Fecha y Hora

```sql
-- Extraer componentes de fecha
SELECT 
    username,
    creado_en,
    EXTRACT(YEAR FROM creado_en) AS año_registro,
    EXTRACT(MONTH FROM creado_en) AS mes_registro,
    EXTRACT(DAY FROM creado_en) AS dia_registro,
    EXTRACT(DOW FROM creado_en) AS dia_semana,
    EXTRACT(HOUR FROM creado_en) AS hora_registro
FROM usuarios;

-- Truncar fechas
SELECT 
    DATE_TRUNC('month', creado_en) AS mes_registro,
    COUNT(*) AS usuarios_registrados
FROM usuarios
GROUP BY DATE_TRUNC('month', creado_en)
ORDER BY mes_registro;

-- Diferencia entre fechas
SELECT 
    s.id,
    u.username,
    s.iniciada_en,
    s.ultima_actividad,
    AGE(s.ultima_actividad, s.iniciada_en) AS duracion_sesion,
    EXTRACT(EPOCH FROM (s.ultima_actividad - s.iniciada_en)) / 60 AS duracion_minutos
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id
WHERE s.ultima_actividad > s.iniciada_en;

-- Sesiones de la última semana
SELECT *
FROM sesiones
WHERE iniciada_en > NOW() - INTERVAL '7 days';
```

### 3.5 Conversión de Tipos

```sql
-- CAST explícito
SELECT 
    m.id,
    CAST(tiempo_ejecucion_ms AS TEXT) AS tiempo_texto,
    CAST(enviado_en AS DATE) AS solo_fecha
FROM mensajes m
JOIN logs_ejecucion l ON m.id = l.mensaje_id;

-- :: sintaxis (PostgreSQL)
SELECT 
    m.id,
    tiempo_ejecucion_ms::TEXT AS tiempo_texto,
    enviado_en::DATE AS solo_fecha
FROM mensajes m
JOIN logs_ejecucion l ON m.id = l.mensaje_id;
```

### 3.6 CASE (Condicionales)

```sql
-- Categorizar usuarios por país
SELECT 
    username,
    pais,
    CASE 
        WHEN pais IN ('Perú', 'Chile', 'Argentina') THEN 'Sudamérica'
        WHEN pais IN ('México', 'Colombia') THEN 'Latinoamérica Norte'
        WHEN pais = 'España' THEN 'Europa'
        ELSE 'Otro'
    END AS region
FROM usuarios;

-- Categorizar tiempos de respuesta
SELECT 
    modulo,
    tiempo_ejecucion_ms,
    CASE 
        WHEN tiempo_ejecucion_ms < 50 THEN 'Rápido'
        WHEN tiempo_ejecucion_ms < 150 THEN 'Normal'
        WHEN tiempo_ejecucion_ms < 300 THEN 'Lento'
        ELSE 'Muy lento'
    END AS categoria_rendimiento
FROM logs_ejecucion
WHERE tiempo_ejecucion_ms IS NOT NULL;
```

---

## 📈 Paso 4: Subconsultas Avanzadas

### 4.1 Subconsultas en SELECT

```sql
-- Mostrar usuarios con conteo de mensajes
SELECT 
    username,
    (SELECT COUNT(*) FROM mensajes WHERE remitente_id = usuarios.id) AS total_mensajes
FROM usuarios
ORDER BY total_mensajes DESC;

-- Mostrar módulos con promedio vs global
SELECT 
    modulo,
    tiempo_ejecucion_ms,
    (SELECT AVG(tiempo_ejecucion_ms) FROM logs_ejecucion) AS promedio_global,
    tiempo_ejecucion_ms - (SELECT AVG(tiempo_ejecucion_ms) FROM logs_ejecucion) AS diferencia
FROM logs_ejecucion;
```

### 4.2 Subconsultas en WHERE

```sql
-- Usuarios con más mensajes que el promedio
SELECT 
    u.username,
    COUNT(m.id) AS total_mensajes
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username
HAVING COUNT(m.id) > (
    SELECT AVG(mensajes_por_usuario)
    FROM (
        SELECT COUNT(*) AS mensajes_por_usuario
        FROM mensajes
        GROUP BY remitente_id
    ) AS subconsulta
);

-- Mensajes con tiempo mayor al promedio de su módulo
SELECT 
    m.id,
    m.contenido,
    l.modulo,
    l.tiempo_ejecucion_ms,
    (SELECT AVG(tiempo_ejecucion_ms) FROM logs_ejecucion l2 WHERE l2.modulo = l.modulo) AS promedio_modulo
FROM mensajes m
JOIN logs_ejecucion l ON m.id = l.mensaje_id
WHERE l.tiempo_ejecucion_ms > (
    SELECT AVG(tiempo_ejecucion_ms) 
    FROM logs_ejecucion l2 
    WHERE l2.modulo = l.modulo
);
```

### 4.3 Subconsultas en FROM (Tablas Derivadas)

```sql
-- Usar subconsulta como tabla
SELECT 
    modulo,
    tiempo_promedio,
    CASE 
        WHEN tiempo_promedio < 100 THEN 'Óptimo'
        WHEN tiempo_promedio < 200 THEN 'Aceptable'
        ELSE 'Revisar'
    END AS estado
FROM (
    SELECT 
        modulo,
        AVG(tiempo_ejecucion_ms) AS tiempo_promedio
    FROM logs_ejecucion
    WHERE exito = TRUE
    GROUP BY modulo
) AS resumen_modulos
ORDER BY tiempo_promedio;

-- Top usuarios por actividad
SELECT 
    username,
    mensajes,
    ranking
FROM (
    SELECT 
        u.username,
        COUNT(m.id) AS mensajes,
        RANK() OVER (ORDER BY COUNT(m.id) DESC) AS ranking
    FROM usuarios u
    LEFT JOIN mensajes m ON u.id = m.remitente_id
    GROUP BY u.id, u.username
) AS ranking_usuarios
WHERE ranking <= 5;
```

### 4.4 Subconsultas Correlacionadas

```sql
-- Usuarios con al menos una sesión en cada canal (ejemplo avanzado)
SELECT DISTINCT u.username
FROM usuarios u
WHERE NOT EXISTS (
    SELECT DISTINCT canal 
    FROM sesiones 
    WHERE canal IS NOT NULL
    EXCEPT
    SELECT DISTINCT s.canal
    FROM sesiones s
    WHERE s.usuario_id = u.id
);

-- Mensajes con sentimiento diferente al promedio de su usuario
SELECT 
    m.id,
    u.username,
    m.contenido,
    (m.metadata_ia->>'sentimiento') AS sentimiento
FROM mensajes m
JOIN usuarios u ON m.remitente_id = u.id
WHERE (m.metadata_ia->>'sentimiento') != (
    SELECT 
        MODE() WITHIN GROUP (ORDER BY (metadata_ia->>'sentimiento'))
    FROM mensajes m2
    WHERE m2.remitente_id = m.remitente_id
);
```

### 4.5 Subconsultas con EXISTS

```sql
-- Usuarios que han enviado al menos un mensaje
SELECT username, email
FROM usuarios u
WHERE EXISTS (
    SELECT 1 FROM mensajes m WHERE m.remitente_id = u.id
);

-- Usuarios con perfil completo y sesiones activas
SELECT username, email
FROM usuarios u
WHERE EXISTS (
    SELECT 1 FROM perfiles_voz p 
    WHERE p.usuario_id = u.id AND p.estado = 'completo'
)
AND EXISTS (
    SELECT 1 FROM sesiones s 
    WHERE s.usuario_id = u.id 
);

-- Módulos que nunca han fallado
SELECT DISTINCT modulo
FROM logs_ejecucion l1
WHERE NOT EXISTS (
    SELECT 1 FROM logs_ejecucion l2 
    WHERE l2.modulo = l1.modulo AND l2.exito = FALSE
);
```

### 4.6 Subconsultas con ANY, ALL

```sql
-- Usuarios con mensajes más largos que cualquier mensaje de otro usuario
SELECT DISTINCT u.username, LENGTH(m.contenido) AS longitud
FROM usuarios u
JOIN mensajes m ON u.id = m.remitente_id
WHERE LENGTH(m.contenido) > ALL (
    SELECT LENGTH(contenido) 
    FROM mensajes 
    WHERE remitente_id != u.id
);

-- Sesiones con más mensajes que alguna sesión de otro usuario
SELECT s.id, u.username, COUNT(m.id) AS mensajes
FROM sesiones s
JOIN usuarios u ON s.usuario_id = u.id
LEFT JOIN mensajes m ON s.id = m.sesion_id
GROUP BY s.id, u.username
HAVING COUNT(m.id) > ANY (
    SELECT COUNT(m2.id)
    FROM sesiones s2
    LEFT JOIN mensajes m2 ON s2.id = m2.sesion_id
    WHERE s2.usuario_id != s.usuario_id
    GROUP BY s2.id
);
```

---

## 🔄 Paso 5: Common Table Expressions (CTEs)

### 5.1 CTE Básica

```sql
-- CTE para contar mensajes por usuario
WITH mensajes_por_usuario AS (
    SELECT 
        u.id,
        u.username,
        COUNT(m.id) AS total_mensajes
    FROM usuarios u
    LEFT JOIN mensajes m ON u.id = m.remitente_id
    GROUP BY u.id, u.username
)
SELECT * FROM mensajes_por_usuario
WHERE total_mensajes > 2
ORDER BY total_mensajes DESC;
```

### 5.2 CTE Múltiples

```sql
-- Análisis completo de actividad
WITH 
-- CTE 1: Resumen de usuarios
usuarios_activos AS (
    SELECT 
        id,
        username,
        pais,
        creado_en
    FROM usuarios
    WHERE activo = TRUE
),
-- CTE 2: Estadísticas de sesiones
estadisticas_sesiones AS (
    SELECT 
        usuario_id,
        COUNT(*) AS total_sesiones,
        AVG(EXTRACT(EPOCH FROM (ultima_actividad - iniciada_en))) / 60 AS duracion_promedio_minutos
    FROM sesiones
    GROUP BY usuario_id
),
-- CTE 3: Estadísticas de mensajes
estadisticas_mensajes AS (
    SELECT 
        remitente_id,
        COUNT(*) AS total_mensajes,
        COUNT(DISTINCT modalidad) AS modalidades_usadas
    FROM mensajes
    GROUP BY remitente_id
)
SELECT 
    ua.username,
    ua.pais,
    ua.creado_en,
    COALESCE(es.total_sesiones, 0) AS sesiones,
    ROUND(COALESCE(es.duracion_promedio_minutos, 0), 2) AS duracion_promedio_min,
    COALESCE(em.total_mensajes, 0) AS mensajes,
    COALESCE(em.modalidades_usadas, 0) AS modalidades
FROM usuarios_activos ua
LEFT JOIN estadisticas_sesiones es ON ua.id = es.usuario_id
LEFT JOIN estadisticas_mensajes em ON ua.id = em.remitente_id
ORDER BY mensajes DESC;
```





