# 📘 Laboratorio 1: ETL

## 🎯 Objetivo

Implementar una arquitectura de datos moderna (Medallón) en Supabase para el caso Aletza, construyendo pipelines ETL/ELT que transformen datos crudos en información valiosa para análisis y modelos de IA.

Al finalizar, podrás:
- Diseñar capas Bronze (Raw), Silver (Cleansed) y Gold (Aggregated)
- Implementar procesos de carga incremental
- Crear pipelines con funciones y procedimientos
- Orquestar transformaciones con SQL
- Preparar datos para consumo en BI y Machine Learning

---

## 📋 Requisitos Previos
- Tener ejecutados los Laboratorios 1-4
- Conectado a Supabase con el SQL Editor abierto
- Tablas originales: `usuarios`, `perfiles_voz`, `sesiones`, `mensajes`, `logs_ejecucion`

---

## 🏗️ Paso 1: Arquitectura Medallón en Supabase

### 1.1 Creación de Esquemas por Capa

```sql
-- Crear esquemas para cada capa de la arquitectura Medallón
CREATE SCHEMA IF NOT EXISTS bronze;
CREATE SCHEMA IF NOT EXISTS silver;
CREATE SCHEMA IF NOT EXISTS gold;

-- Verificar esquemas creados
SELECT schema_name 
FROM information_schema.schemata 
WHERE schema_name IN ('bronze', 'silver', 'gold');
```

### 1.2 Capa Bronze (Raw) - Datos sin Transformar

```sql
-- Tabla Bronze: usuarios_raw (copia exacta de la fuente)
CREATE TABLE bronze.usuarios_raw (
    id INTEGER,
    username TEXT,
    email TEXT,
    creado_en TIMESTAMPTZ,
    pais TEXT,
    ciudad TEXT,
    activo BOOLEAN,
    fecha_carga TIMESTAMPTZ DEFAULT NOW(),
    origen TEXT DEFAULT 'source_system'
);

-- Tabla Bronze: mensajes_raw
CREATE TABLE bronze.mensajes_raw (
    id INTEGER,
    sesion_id INTEGER,
    remitente_id INTEGER,
    contenido TEXT,
    modalidad TEXT,
    metadata_ia JSONB,
    enviado_en TIMESTAMPTZ,
    fecha_carga TIMESTAMPTZ DEFAULT NOW(),
    origen TEXT DEFAULT 'source_system'
);

-- Tabla Bronze: logs_raw
CREATE TABLE bronze.logs_ejecucion_raw (
    id INTEGER,
    mensaje_id INTEGER,
    modulo TEXT,
    tiempo_ejecucion_ms INTEGER,
    exito BOOLEAN,
    error TEXT,
    ejecutado_en TIMESTAMPTZ,
    fecha_carga TIMESTAMPTZ DEFAULT NOW(),
    origen TEXT DEFAULT 'source_system'
);

-- Tabla Bronze: sesiones_raw
CREATE TABLE bronze.sesiones_raw (
    id INTEGER,
    usuario_id INTEGER,
    token TEXT,
    canal TEXT,
    iniciada_en TIMESTAMPTZ,
    ultima_actividad TIMESTAMPTZ,
    expira_en TIMESTAMPTZ,
    cerrada BOOLEAN,
    fecha_carga TIMESTAMPTZ DEFAULT NOW(),
    origen TEXT DEFAULT 'source_system'
);

-- Tabla Bronze: perfiles_voz_raw
CREATE TABLE bronze.perfiles_voz_raw (
    id INTEGER,
    usuario_id INTEGER,
    palabra_magica TEXT,
    vector_voz JSONB,
    muestras_tomadas INTEGER,
    estado TEXT,
    creado_en TIMESTAMPTZ,
    fecha_carga TIMESTAMPTZ DEFAULT NOW(),
    origen TEXT DEFAULT 'source_system'
);

-- Verificar tablas creadas
SELECT table_name, table_schema 
FROM information_schema.tables 
WHERE table_schema = 'bronze'
ORDER BY table_name;
```

### 1.3 Carga Inicial a Bronze (Full Load)

```sql
-- Cargar datos de producción a bronze
INSERT INTO bronze.usuarios_raw (id, username, email, creado_en, pais, ciudad, activo)
SELECT id, username, email, creado_en, pais, ciudad, TRUE
FROM public.usuarios;

INSERT INTO bronze.mensajes_raw (id, sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en)
SELECT id, sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en
FROM public.mensajes;

INSERT INTO bronze.logs_ejecucion_raw (id, mensaje_id, modulo, tiempo_ejecucion_ms, exito, error, ejecutado_en)
SELECT id, mensaje_id, modulo, tiempo_ejecucion_ms, exito, error, ejecutado_en
FROM public.logs_ejecucion;

INSERT INTO bronze.sesiones_raw (id, usuario_id, token, canal, iniciada_en, ultima_actividad, expira_en, cerrada)
SELECT id, usuario_id, token, canal, iniciada_en, ultima_actividad, expira_en, cerrada
FROM public.sesiones;

INSERT INTO bronze.perfiles_voz_raw (id, usuario_id, palabra_magica, vector_voz, muestras_tomadas, estado, creado_en)
SELECT id, usuario_id, palabra_magica, vector_voz, muestras_tomadas, estado, creado_en
FROM public.perfiles_voz;

-- Verificar cargas
SELECT 'bronze.usuarios_raw' AS tabla, COUNT(*) FROM bronze.usuarios_raw
UNION ALL
SELECT 'bronze.mensajes_raw', COUNT(*) FROM bronze.mensajes_raw
UNION ALL
SELECT 'bronze.logs_ejecucion_raw', COUNT(*) FROM bronze.logs_ejecucion_raw;
```



### 1.4 Función para Carga Incremental a Bronze

```sql
-- Función que carga nuevos registros a bronze (incremental)
CREATE OR REPLACE FUNCTION bronze.cargar_incremental_usuarios()
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_nuevos INTEGER;
BEGIN
    -- Insertar usuarios nuevos (que no existen en bronze)
    INSERT INTO bronze.usuarios_raw (id, username, email, creado_en, pais, ciudad, activo)
    SELECT u.id, u.username, u.email, u.creado_en, u.pais, u.ciudad, u.activo
    FROM public.usuarios u
    LEFT JOIN bronze.usuarios_raw b ON u.id = b.id
    WHERE b.id IS NULL;
    
    GET DIAGNOSTICS v_nuevos = ROW_COUNT;
    RETURN v_nuevos;
END;
$$;

-- Probar función
SELECT bronze.cargar_incremental_usuarios();
```

---

## 🧹 Paso 2: Capa Silver (Cleansed) - Datos Limpiados

### 2.1 Crear Tablas Silver

```sql
-- Tabla Silver: usuarios_cleansed (datos limpios y normalizados)
CREATE TABLE silver.usuarios_cleansed (
    id INTEGER PRIMARY KEY,
    username TEXT NOT NULL,
    email TEXT NOT NULL,
    pais TEXT,
    ciudad TEXT,
    activo BOOLEAN DEFAULT TRUE,
    email_valido BOOLEAN,
    dominio_email TEXT,
    fecha_registro DATE,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Silver: mensajes_cleansed
CREATE TABLE silver.mensajes_cleansed (
    id INTEGER PRIMARY KEY,
    sesion_id INTEGER,
    remitente_id INTEGER,
    contenido_limpio TEXT,
    modalidad TEXT,
    intencion TEXT,
    sentimiento TEXT,
    longitud_contenido INTEGER,
    fecha_envio DATE,
    hora_envio TIME,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Silver: logs_cleansed
CREATE TABLE silver.logs_cleansed (
    id INTEGER PRIMARY KEY,
    mensaje_id INTEGER,
    modulo TEXT,
    tiempo_ejecucion_ms INTEGER,
    tiempo_segundos NUMERIC,
    exito BOOLEAN,
    fecha_ejecucion DATE,
    hora_ejecucion TIME,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Silver: sesiones_cleansed
CREATE TABLE silver.sesiones_cleansed (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    canal TEXT,
    fecha_inicio DATE,
    hora_inicio TIME,
    duracion_minutos NUMERIC,
    activa BOOLEAN,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Silver: perfiles_voz_cleansed
CREATE TABLE silver.perfiles_voz_cleansed (
    id INTEGER PRIMARY KEY,
    usuario_id INTEGER,
    estado TEXT,
    muestras_tomadas INTEGER,
    perfil_completo BOOLEAN,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);
```

### 2.2 Procedimiento ETL para Silver (Limpieza y Transformación)

```sql
-- Procedimiento que transforma datos de Bronze a Silver
CREATE OR REPLACE PROCEDURE silver.transformar_usuarios()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Limpiar y transformar datos de usuarios
    INSERT INTO silver.usuarios_cleansed (
        id, username, email, pais, ciudad, activo,
        email_valido, dominio_email, fecha_registro
    )
    SELECT 
        u.id,
        TRIM(u.username) AS username,
        LOWER(TRIM(u.email)) AS email,
        INITCAP(TRIM(u.pais)) AS pais,
        INITCAP(TRIM(u.ciudad)) AS ciudad,
        u.activo,
        u.email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' AS email_valido,
        SPLIT_PART(u.email, '@', 2) AS dominio_email,
        DATE(u.creado_en) AS fecha_registro
    FROM bronze.usuarios_raw u
    ON CONFLICT (id) DO UPDATE SET
        username = EXCLUDED.username,
        email = EXCLUDED.email,
        pais = EXCLUDED.pais,
        ciudad = EXCLUDED.ciudad,
        activo = EXCLUDED.activo,
        email_valido = EXCLUDED.email_valido,
        dominio_email = EXCLUDED.dominio_email,
        fecha_registro = EXCLUDED.fecha_registro,
        fecha_carga = NOW();
    
    RAISE NOTICE 'Usuarios transformados a Silver';
END;
$$;

-- Procedimiento para transformar mensajes
CREATE OR REPLACE PROCEDURE silver.transformar_mensajes()
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO silver.mensajes_cleansed (
        id, sesion_id, remitente_id, contenido_limpio, modalidad,
        intencion, sentimiento, longitud_contenido, fecha_envio, hora_envio
    )
    SELECT 
        m.id,
        m.sesion_id,
        m.remitente_id,
        TRIM(REGEXP_REPLACE(m.contenido, '\s+', ' ', 'g')) AS contenido_limpio,
        m.modalidad,
        m.metadata_ia->>'intencion' AS intencion,
        m.metadata_ia->>'sentimiento' AS sentimiento,
        LENGTH(COALESCE(m.contenido, '')) AS longitud_contenido,
        DATE(m.enviado_en) AS fecha_envio,
        m.enviado_en::TIME AS hora_envio
    FROM bronze.mensajes_raw m
    ON CONFLICT (id) DO UPDATE SET
        contenido_limpio = EXCLUDED.contenido_limpio,
        intencion = EXCLUDED.intencion,
        sentimiento = EXCLUDED.sentimiento,
        longitud_contenido = EXCLUDED.longitud_contenido,
        fecha_carga = NOW();
    
    RAISE NOTICE 'Mensajes transformados a Silver';
END;
$$;

-- Procedimiento para transformar logs
CREATE OR REPLACE PROCEDURE silver.transformar_logs()
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO silver.logs_cleansed (
        id, mensaje_id, modulo, tiempo_ejecucion_ms, tiempo_segundos, exito,
        fecha_ejecucion, hora_ejecucion
    )
    SELECT 
        l.id,
        l.mensaje_id,
        l.modulo,
        l.tiempo_ejecucion_ms,
        ROUND(l.tiempo_ejecucion_ms / 1000.0, 3) AS tiempo_segundos,
        l.exito,
        DATE(l.ejecutado_en) AS fecha_ejecucion,
        l.ejecutado_en::TIME AS hora_ejecucion
    FROM bronze.logs_ejecucion_raw l
    ON CONFLICT (id) DO UPDATE SET
        tiempo_ejecucion_ms = EXCLUDED.tiempo_ejecucion_ms,
        tiempo_segundos = EXCLUDED.tiempo_segundos,
        exito = EXCLUDED.exito,
        fecha_carga = NOW();
    
    RAISE NOTICE 'Logs transformados a Silver';
END;
$$;

-- Ejecutar transformaciones
CALL silver.transformar_usuarios();
CALL silver.transformar_mensajes();
CALL silver.transformar_logs();

-- Verificar datos en Silver
SELECT COUNT(*) FROM silver.usuarios_cleansed;
SELECT COUNT(*) FROM silver.mensajes_cleansed;
SELECT COUNT(*) FROM silver.logs_cleansed;
```

---

## 📊 Paso 3: Capa Gold (Aggregated) - Datos Analíticos

### 3.1 Crear Tablas Gold

```sql
-- Tabla Gold: hechos_mensajes (fact table)
CREATE TABLE gold.hechos_mensajes (
    id SERIAL PRIMARY KEY,
    fecha DATE NOT NULL,
    hora TIME NOT NULL,
    usuario_id INTEGER,
    sesion_id INTEGER,
    modalidad TEXT,
    intencion TEXT,
    sentimiento TEXT,
    longitud_contenido INTEGER,
    tiempo_procesamiento_ms INTEGER,
    modulo_principal TEXT,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Gold: dim_usuarios (dimension table)
CREATE TABLE gold.dim_usuarios (
    usuario_id INTEGER PRIMARY KEY,
    username TEXT,
    pais TEXT,
    ciudad TEXT,
    email_valido BOOLEAN,
    dominio_email TEXT,
    fecha_registro DATE,
    nivel_actividad TEXT,
    total_mensajes INTEGER,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Gold: dim_tiempo (dimension table)
CREATE TABLE gold.dim_tiempo (
    fecha DATE PRIMARY KEY,
    año INTEGER,
    mes INTEGER,
    dia INTEGER,
    dia_semana INTEGER,
    nombre_dia TEXT,
    trimestre INTEGER,
    es_fin_semana BOOLEAN
);

-- Tabla Gold: resumen_diario (aggregated metrics)
CREATE TABLE gold.resumen_diario (
    fecha DATE PRIMARY KEY,
    total_usuarios_activos INTEGER,
    total_sesiones INTEGER,
    total_mensajes INTEGER,
    mensajes_por_modalidad JSONB,
    intenciones_top JSONB,
    tiempo_promedio_respuesta NUMERIC,
    tasa_exito_ia NUMERIC,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Tabla Gold: resumen_modulos (performance metrics)
CREATE TABLE gold.resumen_modulos (
    modulo TEXT PRIMARY KEY,
    total_ejecuciones INTEGER,
    tasa_exito NUMERIC,
    tiempo_promedio_ms NUMERIC,
    tiempo_minimo_ms NUMERIC,
    tiempo_maximo_ms NUMERIC,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);
```

### 3.2 Procedimiento para Poblar Gold

```sql
-- Eliminar si existe
DROP PROCEDURE IF EXISTS gold.actualizar_capa_gold();

-- Procedimiento principal para poblar capa Gold
CREATE OR REPLACE PROCEDURE gold.actualizar_capa_gold()
LANGUAGE plpgsql
AS $$
DECLARE
    v_fecha_inicio DATE;
    v_fecha_fin DATE;
BEGIN
    -- Obtener última fecha procesada
    SELECT COALESCE(MAX(fecha), '2024-01-01') INTO v_fecha_inicio 
    FROM gold.resumen_diario;
    
    v_fecha_fin := CURRENT_DATE - 1;
    
    RAISE NOTICE 'Procesando fechas desde % hasta %', v_fecha_inicio, v_fecha_fin;
    
    -- 1. Actualizar dimensiones - Dim Usuarios
    INSERT INTO gold.dim_usuarios (
        usuario_id, username, pais, ciudad, email_valido, dominio_email,
        fecha_registro, nivel_actividad, total_mensajes
    )
    SELECT 
        u.id,
        u.username,
        u.pais,
        u.ciudad,
        u.email_valido,
        u.dominio_email,
        u.fecha_registro,
        CASE 
            WHEN COALESCE(COUNT(m.id), 0) > 10 THEN 'Alto'
            WHEN COALESCE(COUNT(m.id), 0) > 5 THEN 'Medio'
            ELSE 'Bajo'
        END AS nivel_actividad,
        COUNT(m.id) AS total_mensajes
    FROM silver.usuarios_cleansed u
    LEFT JOIN silver.mensajes_cleansed m ON u.id = m.remitente_id
    GROUP BY u.id, u.username, u.pais, u.ciudad, u.email_valido, u.dominio_email, u.fecha_registro
    ON CONFLICT (usuario_id) DO UPDATE SET
        nivel_actividad = EXCLUDED.nivel_actividad,
        total_mensajes = EXCLUDED.total_mensajes,
        fecha_carga = NOW();
    
    RAISE NOTICE '✅ Dim Usuarios actualizada';
    
    -- 2. Poblar tabla de hechos (solo nuevos registros)
    INSERT INTO gold.hechos_mensajes (
        fecha, hora, usuario_id, sesion_id, modalidad, intencion,
        sentimiento, longitud_contenido, tiempo_procesamiento_ms, modulo_principal
    )
    SELECT 
        m.fecha_envio,
        m.hora_envio,
        m.remitente_id,
        m.sesion_id,
        m.modalidad,
        m.intencion,
        m.sentimiento,
        m.longitud_contenido,
        l.tiempo_ejecucion_ms,
        l.modulo
    FROM silver.mensajes_cleansed m
    LEFT JOIN silver.logs_cleansed l ON m.id = l.mensaje_id
    WHERE m.fecha_envio >= v_fecha_inicio;
    
    RAISE NOTICE '✅ Hechos de mensajes actualizados';
    
    -- 3. Actualizar resumen diario
    INSERT INTO gold.resumen_diario (
        fecha, total_usuarios_activos, total_sesiones, total_mensajes,
        mensajes_por_modalidad, intenciones_top, tiempo_promedio_respuesta, tasa_exito_ia
    )
    WITH 
    -- CTE 1: Estadísticas diarias base
    daily_base AS (
        SELECT 
            m.fecha_envio AS fecha,
            COUNT(DISTINCT m.remitente_id) AS usuarios_activos,
            COUNT(DISTINCT m.sesion_id) AS sesiones,
            COUNT(*) AS mensajes,
            AVG(l.tiempo_ejecucion_ms) AS tiempo_promedio,
            AVG(CASE WHEN l.exito THEN 100 ELSE 0 END) AS tasa_exito
        FROM silver.mensajes_cleansed m
        LEFT JOIN silver.logs_cleansed l ON m.id = l.mensaje_id
        WHERE m.fecha_envio >= v_fecha_inicio
        GROUP BY m.fecha_envio
    ),
    -- CTE 2: Modalidades por día
    modalidades_dia AS (
        SELECT 
            fecha_envio,
            jsonb_object_agg(modalidad, cantidad) AS mensajes_por_modalidad
        FROM (
            SELECT 
                fecha_envio,
                modalidad,
                COUNT(*) AS cantidad
            FROM silver.mensajes_cleansed
            WHERE fecha_envio >= v_fecha_inicio
            GROUP BY fecha_envio, modalidad
        ) sub
        GROUP BY fecha_envio
    ),
    -- CTE 3: Intenciones top por día
    intenciones_top_dia AS (
        SELECT 
            fecha_envio,
            jsonb_agg(intencion ORDER BY cantidad DESC) AS intenciones_top
        FROM (
            SELECT 
                fecha_envio,
                intencion,
                COUNT(*) AS cantidad,
                ROW_NUMBER() OVER (PARTITION BY fecha_envio ORDER BY COUNT(*) DESC) AS rn
            FROM silver.mensajes_cleansed
            WHERE fecha_envio >= v_fecha_inicio
              AND intencion IS NOT NULL
            GROUP BY fecha_envio, intencion
        ) sub
        WHERE rn <= 3
        GROUP BY fecha_envio
    )
    SELECT 
        db.fecha,
        db.usuarios_activos,
        db.sesiones,
        db.mensajes,
        COALESCE(md.mensajes_por_modalidad, '{}'::JSONB) AS mensajes_por_modalidad,
        COALESCE(itd.intenciones_top, '[]'::JSONB) AS intenciones_top,
        ROUND(COALESCE(db.tiempo_promedio, 0), 2) AS tiempo_promedio_respuesta,
        ROUND(COALESCE(db.tasa_exito, 0), 2) AS tasa_exito_ia
    FROM daily_base db
    LEFT JOIN modalidades_dia md ON db.fecha = md.fecha_envio
    LEFT JOIN intenciones_top_dia itd ON db.fecha = itd.fecha_envio
    ON CONFLICT (fecha) DO UPDATE SET
        total_usuarios_activos = EXCLUDED.total_usuarios_activos,
        total_sesiones = EXCLUDED.total_sesiones,
        total_mensajes = EXCLUDED.total_mensajes,
        mensajes_por_modalidad = EXCLUDED.mensajes_por_modalidad,
        intenciones_top = EXCLUDED.intenciones_top,
        tiempo_promedio_respuesta = EXCLUDED.tiempo_promedio_respuesta,
        tasa_exito_ia = EXCLUDED.tasa_exito_ia,
        fecha_carga = NOW();
    
    RAISE NOTICE '✅ Resumen diario actualizado';
    
    -- 4. Actualizar resumen de módulos
    INSERT INTO gold.resumen_modulos (
        modulo, total_ejecuciones, tasa_exito, tiempo_promedio_ms,
        tiempo_minimo_ms, tiempo_maximo_ms
    )
    SELECT 
        modulo,
        COUNT(*) AS total,
        ROUND(100.0 * SUM(CASE WHEN exito THEN 1 ELSE 0 END) / COUNT(*), 2) AS tasa_exito,
        ROUND(AVG(tiempo_ejecucion_ms), 2) AS tiempo_promedio,
        MIN(tiempo_ejecucion_ms) AS tiempo_minimo,
        MAX(tiempo_ejecucion_ms) AS tiempo_maximo
    FROM silver.logs_cleansed
    WHERE fecha_ejecucion >= v_fecha_inicio
    GROUP BY modulo
    ON CONFLICT (modulo) DO UPDATE SET
        total_ejecuciones = EXCLUDED.total_ejecuciones,
        tasa_exito = EXCLUDED.tasa_exito,
        tiempo_promedio_ms = EXCLUDED.tiempo_promedio_ms,
        tiempo_minimo_ms = EXCLUDED.tiempo_minimo_ms,
        tiempo_maximo_ms = EXCLUDED.tiempo_maximo_ms,
        fecha_carga = NOW();
    
    RAISE NOTICE '✅ Resumen de módulos actualizado';
    RAISE NOTICE '🎯 Capa Gold actualizada exitosamente para fechas desde % hasta %', v_fecha_inicio, v_fecha_fin;
    
EXCEPTION WHEN OTHERS THEN
    RAISE NOTICE '❌ Error en actualización de Gold: %', SQLERRM;
    RAISE;
END;
$$;

```

---

## ⏰ Paso 4: Carga Incremental y CDC

### 4.1 Marca de Agua (Watermark) para Incremental

```sql
-- Crear tabla de control de cargas
CREATE TABLE IF NOT EXISTS control_cargas (
    id SERIAL PRIMARY KEY,
    proceso TEXT NOT NULL,
    ultima_ejecucion TIMESTAMPTZ,
    ultimo_id_procesado INTEGER,
    registros_procesados INTEGER,
    estado TEXT,
    fecha_carga TIMESTAMPTZ DEFAULT NOW()
);

-- Función para registrar ejecuciones
CREATE OR REPLACE FUNCTION registrar_ejecucion(
    p_proceso TEXT,
    p_ultimo_id INTEGER,
    p_registros INTEGER,
    p_estado TEXT
)
RETURNS VOID
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO control_cargas (proceso, ultima_ejecucion, ultimo_id_procesado, registros_procesados, estado)
    VALUES (p_proceso, NOW(), p_ultimo_id, p_registros, p_estado);
    
    RAISE NOTICE 'Ejecución registrada: % - % registros', p_proceso, p_registros;
END;
$$;
```

### 4.2 Pipeline Completo con Carga Incremental

```sql
-- Pipeline completo: Bronze → Silver → Gold
CREATE OR REPLACE PROCEDURE pipeline_completo_aletza()
LANGUAGE plpgsql
AS $$
DECLARE
    v_nuevos_usuarios INTEGER;
    v_nuevos_mensajes INTEGER;
    v_nuevos_logs INTEGER;
BEGIN
    -- 1. Cargar datos nuevos a Bronze
    RAISE NOTICE 'Iniciando carga a Bronze...';
    
    v_nuevos_usuarios := bronze.cargar_incremental_usuarios();
    -- Nota: Similar para otras tablas (implementar según necesidad)
    
    -- 2. Transformar a Silver
    RAISE NOTICE 'Transformando a Silver...';
    CALL silver.transformar_usuarios();
    CALL silver.transformar_mensajes();
    CALL silver.transformar_logs();
    
    -- 3. Actualizar Gold
    RAISE NOTICE 'Actualizando Gold...';
    CALL gold.actualizar_capa_gold();
    
    -- 4. Registrar ejecución
    PERFORM registrar_ejecucion('pipeline_completo', NULL, 
                                v_nuevos_usuarios + v_nuevos_mensajes + v_nuevos_logs, 'EXITOSO');
    
    RAISE NOTICE 'Pipeline completado exitosamente. Nuevos registros: %', 
                 v_nuevos_usuarios + v_nuevos_mensajes + v_nuevos_logs;
END;
$$;

-- Ejecutar pipeline completo
CALL pipeline_completo_aletza();
```
