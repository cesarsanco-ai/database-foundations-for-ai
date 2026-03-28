# 📘 Laboratorio 2: Programación en el Servidor

## 🔄 Paso 3: Procedimientos Almacenados

### 3.1 Procedimiento para Limpiar Sesiones Expiradas

```sql
ALTER TABLE sesiones
ADD COLUMN cerrada BOOLEAN DEFAULT FALSE;

-- Procedimiento que cierra sesiones expiradas
CREATE OR REPLACE PROCEDURE limpiar_sesiones_expiradas()
LANGUAGE plpgsql
AS $$
DECLARE
    v_actualizadas INTEGER;
BEGIN
    UPDATE sesiones
    SET cerrada = TRUE
    WHERE expira_en < NOW() 
      AND cerrada = FALSE;
    
    GET DIAGNOSTICS v_actualizadas = ROW_COUNT;
    
    RAISE NOTICE 'Sesiones cerradas: %', v_actualizadas;
END;
$$;

-- Ejecutar procedimiento
CALL limpiar_sesiones_expiradas();
```

### 3.2 Procedimiento para Generar Reporte Diario

```sql
-- Crear tabla de reportes diarios
CREATE TABLE IF NOT EXISTS reportes_diarios (
    id SERIAL PRIMARY KEY,
    fecha DATE NOT NULL UNIQUE,
    total_usuarios_activos INTEGER,
    total_sesiones INTEGER,
    total_mensajes INTEGER,
    mensajes_por_modalidad JSONB,
    modulo_mas_usado TEXT,
    tasa_exito_promedio NUMERIC,
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Procedimiento para generar reporte diario
CREATE OR REPLACE PROCEDURE generar_reporte_diario(p_fecha DATE DEFAULT CURRENT_DATE - 1)
LANGUAGE plpgsql
AS $$
DECLARE
    v_total_usuarios INTEGER;
    v_total_sesiones INTEGER;
    v_total_mensajes INTEGER;
    v_mensajes_por_modalidad JSONB;
    v_modulo_mas_usado TEXT;
    v_tasa_exito_promedio NUMERIC;
BEGIN
    -- Calcular métricas
    SELECT COUNT(DISTINCT usuario_id) INTO v_total_usuarios
    FROM sesiones
    WHERE DATE(iniciada_en) = p_fecha;
    
    SELECT COUNT(*) INTO v_total_sesiones
    FROM sesiones
    WHERE DATE(iniciada_en) = p_fecha;
    
    SELECT COUNT(*) INTO v_total_mensajes
    FROM mensajes
    WHERE DATE(enviado_en) = p_fecha;
    
    -- Modalidades
    SELECT jsonb_object_agg(modalidad, total) INTO v_mensajes_por_modalidad
    FROM (
        SELECT modalidad, COUNT(*) AS total
        FROM mensajes
        WHERE DATE(enviado_en) = p_fecha
        GROUP BY modalidad
    ) sub;
    
    -- Módulo más usado
    SELECT modulo INTO v_modulo_mas_usado
    FROM logs_ejecucion l
    JOIN mensajes m ON l.mensaje_id = m.id
    WHERE DATE(m.enviado_en) = p_fecha
    GROUP BY modulo
    ORDER BY COUNT(*) DESC
    LIMIT 1;
    
    -- Tasa de éxito promedio
    SELECT AVG(CASE WHEN exito THEN 100 ELSE 0 END) INTO v_tasa_exito_promedio
    FROM logs_ejecucion l
    JOIN mensajes m ON l.mensaje_id = m.id
    WHERE DATE(m.enviado_en) = p_fecha;
    
    -- Insertar o actualizar reporte
    INSERT INTO reportes_diarios (
        fecha,
        total_usuarios_activos,
        total_sesiones,
        total_mensajes,
        mensajes_por_modalidad,
        modulo_mas_usado,
        tasa_exito_promedio
    ) VALUES (
        p_fecha,
        v_total_usuarios,
        v_total_sesiones,
        v_total_mensajes,
        COALESCE(v_mensajes_por_modalidad, '{}'::JSONB),
        v_modulo_mas_usado,
        ROUND(v_tasa_exito_promedio, 2)
    )
    ON CONFLICT (fecha) DO UPDATE SET
        total_usuarios_activos = EXCLUDED.total_usuarios_activos,
        total_sesiones = EXCLUDED.total_sesiones,
        total_mensajes = EXCLUDED.total_mensajes,
        mensajes_por_modalidad = EXCLUDED.mensajes_por_modalidad,
        modulo_mas_usado = EXCLUDED.modulo_mas_usado,
        tasa_exito_promedio = EXCLUDED.tasa_exito_promedio;
    
    RAISE NOTICE 'Reporte generado para fecha: %', p_fecha;
END;
$$;

-- Ejecutar procedimiento
CALL generar_reporte_diario('2024-01-15');
CALL generar_reporte_diario(); -- Usa fecha por defecto (ayer)

-- Verificar reportes
SELECT * FROM reportes_diarios ORDER BY fecha DESC;
```

### 3.3 Procedimiento con Parámetros de Salida

```sql
-- Procedimiento con parámetros INOUT
CREATE OR REPLACE PROCEDURE obtener_estadisticas_usuario(
    IN p_usuario_id INTEGER,
    OUT p_total_mensajes INTEGER,
    OUT p_total_sesiones INTEGER,
    OUT p_ultima_actividad TIMESTAMPTZ,
    OUT p_nivel_actividad TEXT
)
LANGUAGE plpgsql
AS $$
BEGIN
    SELECT COUNT(*) INTO p_total_mensajes
    FROM mensajes
    WHERE remitente_id = p_usuario_id;
    
    SELECT COUNT(*) INTO p_total_sesiones
    FROM sesiones
    WHERE usuario_id = p_usuario_id;
    
    SELECT MAX(iniciada_en) INTO p_ultima_actividad
    FROM sesiones
    WHERE usuario_id = p_usuario_id;
    
    -- Calcular nivel
    IF p_total_mensajes > 10 AND p_total_sesiones > 3 THEN
        p_nivel_actividad := 'Alto';
    ELSIF p_total_mensajes > 5 OR p_total_sesiones > 1 THEN
        p_nivel_actividad := 'Medio';
    ELSE
        p_nivel_actividad := 'Bajo';
    END IF;
END;
$$;

-- Llamar procedimiento con OUT
DO $$
DECLARE
    v_mensajes INTEGER;
    v_sesiones INTEGER;
    v_ultima TIMESTAMPTZ;
    v_nivel TEXT;
BEGIN
    CALL obtener_estadisticas_usuario(1, v_mensajes, v_sesiones, v_ultima, v_nivel);
    RAISE NOTICE 'Usuario 1: Mensajes=%, Sesiones=%, Última=%, Nivel=%', 
                 v_mensajes, v_sesiones, v_ultima, v_nivel;
END $$;
```

---

## ⚡ Paso 4: Triggers (Disparadores)

### 4.1 Trigger para Actualizar Última Actividad

```sql
-- Función para actualizar última actividad en sesión
CREATE OR REPLACE FUNCTION actualizar_ultima_actividad()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE sesiones
    SET ultima_actividad = NEW.enviado_en
    WHERE id = NEW.sesion_id;
    
    RETURN NEW;
END;
$$;

-- Crear trigger en mensajes
CREATE TRIGGER trig_actualizar_actividad
AFTER INSERT ON mensajes
FOR EACH ROW
EXECUTE FUNCTION actualizar_ultima_actividad();

-- Probar trigger
INSERT INTO mensajes (sesion_id, remitente_id, contenido, modalidad)
VALUES (3, 1, 'Mensaje de prueba', 'texto');

-- Verificar que se actualizó ultima_actividad
SELECT id, ultima_actividad FROM sesiones WHERE id = 3;
```

### 4.2 Trigger para Auditoría de Cambios en Usuarios

```sql
-- Crear tabla de auditoría
CREATE TABLE IF NOT EXISTS auditoria_usuarios (
    id SERIAL PRIMARY KEY,
    usuario_id INTEGER,
    campo_modificado TEXT,
    valor_anterior TEXT,
    valor_nuevo TEXT,
    fecha TIMESTAMPTZ DEFAULT NOW(),
    usuario TEXT DEFAULT current_user
);

-- Función de auditoría
CREATE OR REPLACE FUNCTION auditar_cambios_usuarios()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Auditar cambios en email
    IF OLD.email IS DISTINCT FROM NEW.email THEN
        INSERT INTO auditoria_usuarios (usuario_id, campo_modificado, valor_anterior, valor_nuevo)
        VALUES (OLD.id, 'email', OLD.email, NEW.email);
    END IF;
    
    -- Auditar cambios en país
    IF OLD.pais IS DISTINCT FROM NEW.pais THEN
        INSERT INTO auditoria_usuarios (usuario_id, campo_modificado, valor_anterior, valor_nuevo)
        VALUES (OLD.id, 'pais', OLD.pais, NEW.pais);
    END IF;
    
    -- Auditar cambios en activo
    IF OLD.activo IS DISTINCT FROM NEW.activo THEN
        INSERT INTO auditoria_usuarios (usuario_id, campo_modificado, valor_anterior, valor_nuevo)
        VALUES (OLD.id, 'activo', OLD.activo::TEXT, NEW.activo::TEXT);
    END IF;
    
    RETURN NEW;
END;
$$;

-- Crear trigger en usuarios
CREATE TRIGGER trig_auditar_usuarios
AFTER UPDATE ON usuarios
FOR EACH ROW
EXECUTE FUNCTION auditar_cambios_usuarios();
```

### 4.3 Trigger BEFORE para Validación

```sql
-- Trigger para validar que el token de sesión sea único
CREATE OR REPLACE FUNCTION validar_token_sesion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Verificar que token no exista
    IF EXISTS (SELECT 1 FROM sesiones WHERE token = NEW.token AND id != COALESCE(OLD.id, -1)) THEN
        RAISE EXCEPTION 'Token % ya existe', NEW.token;
    END IF;
    
    -- Validar formato del token (al menos 5 caracteres)
    IF LENGTH(NEW.token) < 5 THEN
        RAISE EXCEPTION 'Token debe tener al menos 5 caracteres';
    END IF;
    
    RETURN NEW;
END;
$$;

-- Crear trigger
CREATE TRIGGER trig_validar_token
BEFORE INSERT OR UPDATE ON sesiones
FOR EACH ROW
EXECUTE FUNCTION validar_token_sesion();

```

### 4.4 Trigger INSTEAD OF en Vista

```sql
-- Crear vista de mensajes resumidos
CREATE VIEW vista_mensajes_resumen AS
SELECT 
    m.id,
    u.username,
    m.contenido,
    m.modalidad,
    m.enviado_en
FROM mensajes m
JOIN usuarios u ON m.remitente_id = u.id;

-- Función para INSTEAD OF INSERT en vista (PostgreSQL requiere función)
-- Nota: PostgreSQL no soporta INSTEAD OF triggers en vistas directamente
-- Usamos RULE en su lugar
CREATE OR REPLACE RULE vista_mensajes_resumen_insert AS
ON INSERT TO vista_mensajes_resumen
DO INSTEAD
INSERT INTO mensajes (sesion_id, remitente_id, contenido, modalidad, enviado_en)
VALUES (
    (SELECT id FROM sesiones WHERE usuario_id = (SELECT id FROM usuarios WHERE username = NEW.username) LIMIT 1),
    (SELECT id FROM usuarios WHERE username = NEW.username),
    NEW.contenido,
    NEW.modalidad,
    NEW.enviado_en
);

```

### 4.5 Trigger con Condiciones (WHEN)

```sql
-- Trigger solo para mensajes de audio (para procesamiento especial)
CREATE OR REPLACE FUNCTION procesar_mensaje_audio()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Insertar log especial para mensajes de audio
    INSERT INTO logs_ejecucion (mensaje_id, modulo, tiempo_ejecucion_ms, exito)
    VALUES (NEW.id, 'procesamiento_audio', 0, TRUE);
    
    RAISE NOTICE 'Mensaje de audio recibido: ID %', NEW.id;
    RETURN NEW;
END;
$$;

-- Crear trigger con condición WHEN (PostgreSQL 9.6+)
CREATE TRIGGER trig_procesar_audio
AFTER INSERT ON mensajes
FOR EACH ROW
WHEN (NEW.modalidad = 'audio')
EXECUTE FUNCTION procesar_mensaje_audio();

-- Probar con mensaje de audio
INSERT INTO mensajes (sesion_id, remitente_id, contenido, modalidad)
VALUES (3, 1, 'Grabación de prueba', 'audio');
```
 
---

## 🔐 Paso 5: Seguridad y Permisos

### 5.1 Crear Roles y Usuarios

```sql
-- NOTA: En Supabase, los roles se gestionan a través de auth.users
-- Pero podemos crear roles dentro de la base de datos

-- Crear rol de aplicación
CREATE ROLE app_role;

-- Crear rol de analista
CREATE ROLE analista_role;

-- Crear rol de administrador
CREATE ROLE admin_role;

-- Asignar permisos básicos
GRANT CONNECT ON DATABASE postgres TO app_role, analista_role, admin_role;
GRANT USAGE ON SCHEMA public TO app_role, analista_role, admin_role;

-- Permisos para app_role (solo lectura de vistas)
GRANT SELECT ON vista_resumen_actividad TO app_role;
GRANT SELECT ON vista_rendimiento_modulos TO app_role;

-- Permisos para analista_role (lectura de tablas principales)
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analista_role;

-- Permisos para admin_role (todo)
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin_role;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO admin_role;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO admin_role;
```

### 5.2 Row Level Security (RLS)

```sql
-- Habilitar RLS en usuarios
ALTER TABLE usuarios ENABLE ROW LEVEL SECURITY;

-- Política: Usuarios solo ven sus propios datos
CREATE POLICY usuarios_self_policy ON usuarios
    FOR SELECT
    USING (current_user = username);

-- Política: Admins pueden ver todo
CREATE POLICY usuarios_admin_policy ON usuarios
    FOR ALL
    USING (current_user IN ('admin', 'supabase_admin'));

-- Ver políticas creadas
SELECT * FROM pg_policies WHERE tablename = 'usuarios';
```

### 5.3 Función con SECURITY DEFINER

```sql
-- Función que ejecuta con permisos del creador
CREATE OR REPLACE FUNCTION eliminar_usuario_seguro(p_username TEXT)
RETURNS VOID
SECURITY DEFINER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Eliminar perfil de voz
    DELETE FROM perfiles_voz 
    WHERE usuario_id = (SELECT id FROM usuarios WHERE username = p_username);
    
    -- Eliminar sesiones
    DELETE FROM sesiones 
    WHERE usuario_id = (SELECT id FROM usuarios WHERE username = p_username);
    
    -- Eliminar mensajes (cascada)
    DELETE FROM mensajes 
    WHERE remitente_id = (SELECT id FROM usuarios WHERE username = p_username);
    
    -- Eliminar usuario
    DELETE FROM usuarios WHERE username = p_username;
    
    RAISE NOTICE 'Usuario % eliminado correctamente', p_username;
END;
$$;

-- Otorgar ejecución a app_role
GRANT EXECUTE ON FUNCTION eliminar_usuario_seguro(TEXT) TO app_role;
```

---

## 📊 Paso 6: Ejercicios Prácticos Integradores

### Ejercicio 1: Vista de Dashboard

```sql
-- Crear una vista que muestre un dashboard con:
-- - Total de usuarios
-- - Usuarios con perfil completo
-- - Total de mensajes por día
-- - Módulo más utilizado
-- - Tasa de éxito general

CREATE OR REPLACE VIEW vista_dashboard_aletza AS
WITH 
total_usuarios AS (
    SELECT COUNT(*) AS total FROM usuarios WHERE activo = TRUE
),
usuarios_perfil_completo AS (
    SELECT COUNT(*) AS total FROM perfiles_voz WHERE estado = 'completo'
),
mensajes_dia AS (
    SELECT 
        DATE(enviado_en) AS fecha,
        COUNT(*) AS mensajes
    FROM mensajes
    WHERE enviado_en > CURRENT_DATE - INTERVAL '7 days'
    GROUP BY DATE(enviado_en)
),
modulo_mas_usado AS (
    SELECT 
        modulo,
        COUNT(*) AS usos
    FROM logs_ejecucion
    WHERE ejecutado_en > CURRENT_DATE - INTERVAL '7 days'
    GROUP BY modulo
    ORDER BY usos DESC
    LIMIT 1
),
tasa_exito_general AS (
    SELECT 
        ROUND(100.0 * SUM(CASE WHEN exito THEN 1 ELSE 0 END) / COUNT(*), 2) AS tasa
    FROM logs_ejecucion
    WHERE ejecutado_en > CURRENT_DATE - INTERVAL '7 days'
)
SELECT 
    (SELECT total FROM total_usuarios) AS total_usuarios,
    (SELECT total FROM usuarios_perfil_completo) AS usuarios_con_voz,
    (SELECT jsonb_agg(jsonb_build_object('fecha', fecha, 'mensajes', mensajes)) FROM mensajes_dia) AS mensajes_ultimos_7_dias,
    (SELECT modulo FROM modulo_mas_usado) AS modulo_mas_usado,
    (SELECT tasa FROM tasa_exito_general) AS tasa_exito_general;

-- Consultar vista
SELECT * FROM vista_dashboard_aletza;
```

### Ejercicio 2: Función para Recomendaciones

```sql
-- Función que recomienda módulos basados en historial
CREATE OR REPLACE FUNCTION recomendar_modulos(p_usuario_id INTEGER)
RETURNS TABLE(modulo TEXT, probabilidad NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    WITH historial_usuario AS (
        SELECT l.modulo, COUNT(*) AS usos
        FROM logs_ejecucion l
        JOIN mensajes m ON l.mensaje_id = m.id
        WHERE m.remitente_id = p_usuario_id
        GROUP BY l.modulo
    ),
    historial_global AS (
        SELECT l.modulo, COUNT(*) AS usos_globales
        FROM logs_ejecucion l
        GROUP BY l.modulo
    )
    SELECT 
        hg.modulo,
        ROUND(COALESCE(hu.usos, 0)::NUMERIC / hg.usos_globales * 100, 2) AS probabilidad
    FROM historial_global hg
    LEFT JOIN historial_usuario hu ON hg.modulo = hu.modulo
    ORDER BY probabilidad DESC
    LIMIT 5;
END;
$$;

-- Probar función
SELECT * FROM recomendar_modulos(1);
```

### Ejercicio 3: Trigger para Mantener Integridad

```sql
-- Trigger que impide eliminar usuarios con sesiones activas
CREATE OR REPLACE FUNCTION evitar_eliminar_usuario_activo()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_sesiones_activas INTEGER;
BEGIN
    SELECT COUNT(*) INTO v_sesiones_activas
    FROM sesiones
    WHERE usuario_id = OLD.id AND cerrada = FALSE;
    
    IF v_sesiones_activas > 0 THEN
        RAISE EXCEPTION 'No se puede eliminar usuario % porque tiene % sesiones activas', 
                        OLD.username, v_sesiones_activas;
    END IF;
    
    RETURN OLD;
END;
$$;

-- Crear trigger
CREATE TRIGGER trig_evitar_eliminar_usuario_activo
BEFORE DELETE ON usuarios
FOR EACH ROW
EXECUTE FUNCTION evitar_eliminar_usuario_activo();

-- Probar (debería fallar si tiene sesiones activas)
-- DELETE FROM usuarios WHERE id = 1;
```


---

## 📝 Resumen de Conceptos Aplicados

| Objeto | Propósito | Ejemplos en Aletza |
|--------|-----------|-------------------|
| **Vistas** | Simplificar consultas, seguridad | vista_usuarios_activos, vista_rendimiento_modulos |
| **Vistas Materializadas** | Reportes precalculados | mv_resumen_diario |
| **Funciones Escalares** | Cálculos específicos | calcular_nivel_actividad(), tasa_exito_modulo() |
| **Funciones Tabla** | Datos estructurados | mensajes_usuario(), estadisticas_sesiones() |
| **Procedimientos** | Procesos batch | limpiar_sesiones_expiradas(), generar_reporte_diario() |
| **Triggers** | Automatización, auditoría | actualizar_ultima_actividad, auditar_cambios_usuarios |
| **RLS** | Seguridad por fila | Políticas de acceso a usuarios |

---

## 🎯 Conclusión

La programación en el servidor para el caso Aletza permite:

1. **Centralizar reglas de negocio** en la base de datos
2. **Automatizar procesos** con triggers y procedimientos
3. **Mejorar la seguridad** con vistas, RLS y SECURITY DEFINER
4. **Optimizar rendimiento** moviendo lógica cerca de los datos
5. **Facilitar mantenimiento** con código reutilizable

