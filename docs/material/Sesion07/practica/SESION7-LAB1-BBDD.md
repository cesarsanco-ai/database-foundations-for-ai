# 📘 Laboratorio 1: Programación en el Servidor

## 🔍 Paso 1: Vistas (Views) en Supabase

### 1.1 Vista Simple para Usuarios Activos

```sql
-- Crear vista para usuarios activos
CREATE VIEW vista_usuarios_activos AS
SELECT 
    id,
    username,
    email,
    pais,
    ciudad,
    creado_en
FROM usuarios
WHERE activo = TRUE;

-- Consultar la vista
SELECT * FROM vista_usuarios_activos;

```

### 1.2 Vista con JOIN para Resumen de Actividad

```sql
-- Vista que combina usuarios con estadísticas de sesiones y mensajes
CREATE VIEW vista_resumen_actividad AS
SELECT 
    u.id,
    u.username,
    u.email,
    u.pais,
    COUNT(DISTINCT s.id) AS total_sesiones,
    COUNT(DISTINCT m.id) AS total_mensajes,
    MIN(s.iniciada_en) AS primera_sesion,
    MAX(s.iniciada_en) AS ultima_sesion,
    COUNT(DISTINCT m.modalidad) AS modalidades_usadas
FROM usuarios u
LEFT JOIN sesiones s ON u.id = s.usuario_id
LEFT JOIN mensajes m ON u.id = m.remitente_id
WHERE u.activo = TRUE
GROUP BY u.id, u.username, u.email, u.pais;

-- Consultar la vista
SELECT * FROM vista_resumen_actividad ORDER BY total_mensajes DESC;
```

### 1.3 Vista con Datos Sensibles Protegidos

```sql
-- Vista que oculta información sensible según rol
-- NOTA: En Supabase, se usa Row Level Security (RLS) en lugar de vistas para seguridad
-- Pero podemos crear una vista con CASE para simular

CREATE VIEW vista_usuarios_segura AS
SELECT 
    id,
    username,
    email,
    -- Ocultar email parcialmente para usuarios no autenticados
    CASE 
        WHEN current_user = 'authenticated' THEN email
        ELSE '***@' || SPLIT_PART(email, '@', 2)
    END AS email_parcial,
    pais,
    ciudad,
    creado_en
FROM usuarios;

-- Consultar como usuario anónimo (simulado)
SELECT * FROM vista_usuarios_segura;
```

### 1.4 Vista para Análisis de Rendimiento por Módulo

```sql
-- Vista que muestra métricas de rendimiento de módulos IA
CREATE VIEW vista_rendimiento_modulos AS
SELECT 
    l.modulo,
    COUNT(*) AS total_ejecuciones,
    SUM(CASE WHEN l.exito THEN 1 ELSE 0 END) AS exitos,
    ROUND(100.0 * SUM(CASE WHEN l.exito THEN 1 ELSE 0 END) / COUNT(*), 2) AS tasa_exito,
    ROUND(AVG(l.tiempo_ejecucion_ms), 2) AS tiempo_promedio_ms,
    ROUND(MIN(l.tiempo_ejecucion_ms), 2) AS tiempo_minimo_ms,
    ROUND(MAX(l.tiempo_ejecucion_ms), 2) AS tiempo_maximo_ms,
    COUNT(DISTINCT l.mensaje_id) AS mensajes_procesados
FROM logs_ejecucion l
GROUP BY l.modulo
ORDER BY tasa_exito DESC;

-- Consultar la vista
SELECT * FROM vista_rendimiento_modulos;
```

### 1.5 Vista Materializada para Reportes

```sql
-- Vista materializada para reportes diarios (almacena físicamente)
CREATE MATERIALIZED VIEW mv_resumen_diario AS
SELECT 
    DATE(enviado_en) AS fecha,
    modalidad,
    COUNT(*) AS total_mensajes,
    COUNT(DISTINCT remitente_id) AS usuarios_activos
FROM mensajes
WHERE enviado_en >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE(enviado_en), modalidad
ORDER BY fecha DESC, modalidad;

-- Consultar vista materializada
SELECT * FROM mv_resumen_diario;

-- Actualizar vista materializada (ejecutar periódicamente)
REFRESH MATERIALIZED VIEW mv_resumen_diario;

-- Verificar última actualización
SELECT 
    matviewname,
    pg_size_pretty(pg_total_relation_size('mv_resumen_diario'::regclass)) AS tamaño
FROM pg_matviews
WHERE matviewname = 'mv_resumen_diario';
```



---

## ⚙️ Paso 2: Funciones en PostgreSQL

### 2.1 Función Escalar - Calcular Nivel de Actividad

```sql
-- Función que calcula nivel de actividad de un usuario
CREATE OR REPLACE FUNCTION calcular_nivel_actividad(p_usuario_id INTEGER)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    v_total_mensajes INTEGER;
    v_total_sesiones INTEGER;
    v_nivel TEXT;
BEGIN
    -- Contar mensajes del usuario
    SELECT COUNT(*) INTO v_total_mensajes
    FROM mensajes
    WHERE remitente_id = p_usuario_id;
    
    -- Contar sesiones del usuario
    SELECT COUNT(*) INTO v_total_sesiones
    FROM sesiones
    WHERE usuario_id = p_usuario_id;
    
    -- Determinar nivel
    IF v_total_mensajes > 10 AND v_total_sesiones > 3 THEN
        v_nivel := 'Alto';
    ELSIF v_total_mensajes > 5 OR v_total_sesiones > 1 THEN
        v_nivel := 'Medio';
    ELSE
        v_nivel := 'Bajo';
    END IF;
    
    RETURN v_nivel;
END;
$$;

-- Probar función
SELECT 
    username,
    calcular_nivel_actividad(id) AS nivel_actividad
FROM usuarios;
```

### 2.2 Función Escalar - Tasa de Éxito por Módulo

```sql
-- Función que calcula tasa de éxito de un módulo
CREATE OR REPLACE FUNCTION tasa_exito_modulo(p_modulo TEXT)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
    v_exitos INTEGER;
    v_tasa NUMERIC;
BEGIN
    SELECT 
        COUNT(*),
        COUNT(*) FILTER (WHERE exito = TRUE)
    INTO v_total, v_exitos
    FROM logs_ejecucion
    WHERE modulo = p_modulo;
    
    IF v_total = 0 THEN
        RETURN NULL;
    END IF;
    
    v_tasa := (v_exitos::NUMERIC / v_total) * 100;
    RETURN ROUND(v_tasa, 2);
END;
$$;

-- Probar función
SELECT 
    DISTINCT modulo,
    tasa_exito_modulo(modulo) AS tasa_exito
FROM logs_ejecucion
ORDER BY tasa_exito DESC;
```

### 2.3 Función Tabla - Mensajes por Usuario

```sql
-- Función que retorna tabla con mensajes de un usuario
CREATE OR REPLACE FUNCTION mensajes_usuario(p_usuario_id INTEGER)
RETURNS TABLE(
    mensaje_id INTEGER,
    contenido TEXT,
    modalidad TEXT,
    enviado_en TIMESTAMPTZ,
    intencion TEXT,
    sentimiento TEXT
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        m.id,
        m.contenido,
        m.modalidad,
        m.enviado_en,
        m.metadata_ia->>'intencion' AS intencion,
        m.metadata_ia->>'sentimiento' AS sentimiento
    FROM mensajes m
    WHERE m.remitente_id = p_usuario_id
    ORDER BY m.enviado_en DESC;
END;
$$;

-- Probar función
SELECT * FROM mensajes_usuario(1) LIMIT 5;
```

### 2.4 Función Tabla - Estadísticas de Sesión

```sql
-- Función que retorna estadísticas de sesiones por período
CREATE OR REPLACE FUNCTION estadisticas_sesiones(
    p_fecha_inicio DATE,
    p_fecha_fin DATE
)
RETURNS TABLE(
    fecha DATE,
    total_sesiones BIGINT,
    usuarios_unicos BIGINT,
    promedio_duracion_minutos NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        DATE(s.iniciada_en) AS fecha,
        COUNT(*) AS total_sesiones,
        COUNT(DISTINCT s.usuario_id) AS usuarios_unicos,
        ROUND(AVG(EXTRACT(EPOCH FROM (s.ultima_actividad - s.iniciada_en)) / 60), 2) AS promedio_duracion_minutos
    FROM sesiones s
    WHERE DATE(s.iniciada_en) BETWEEN p_fecha_inicio AND p_fecha_fin
    GROUP BY DATE(s.iniciada_en)
    ORDER BY fecha;
END;
$$;

-- Probar función
SELECT * FROM estadisticas_sesiones('2024-01-15', '2024-01-17');
```

### 2.5 Función con Manejo de Excepciones

```sql
-- Función que genera ticket con manejo de errores
CREATE OR REPLACE FUNCTION generar_ticket_seguro(
    p_usuario_id INTEGER,
    p_contenido TEXT
)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_sesion_id INTEGER;
    v_mensaje_id INTEGER;
BEGIN
    -- Buscar sesión activa del usuario
    SELECT id INTO v_sesion_id
    FROM sesiones
    WHERE usuario_id = p_usuario_id 
      AND cerrada = FALSE 
      AND expira_en > NOW()
    ORDER BY iniciada_en DESC
    LIMIT 1;
    
    IF v_sesion_id IS NULL THEN
        RAISE EXCEPTION 'No hay sesión activa para el usuario %', p_usuario_id;
    END IF;
    
    -- Insertar mensaje
    INSERT INTO mensajes (sesion_id, remitente_id, contenido, modalidad)
    VALUES (v_sesion_id, p_usuario_id, p_contenido, 'texto')
    RETURNING id INTO v_mensaje_id;
    
    -- Registrar log
    INSERT INTO logs_ejecucion (mensaje_id, modulo, tiempo_ejecucion_ms, exito)
    VALUES (v_mensaje_id, 'generacion_ticket', 10, TRUE);
    
    RETURN v_mensaje_id;
    
EXCEPTION
    WHEN OTHERS THEN
        -- Registrar error en log
        RAISE NOTICE 'Error en generar_ticket_seguro: %', SQLERRM;
        RETURN NULL;
END;
$$;

-- Probar función
SELECT generar_ticket_seguro(1, 'Mi primer ticket');
```


