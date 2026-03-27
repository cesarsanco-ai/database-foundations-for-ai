# 📘 Laboratorio 1: DDL (Data Definition Language) - Caso Aletza para Supabase

## 📋 Paso 1: Creación de Tablas (DDL Básico)

### 1.1 Tabla de Usuarios

```sql
-- Crear tabla usuarios
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Comentarios para documentación
COMMENT ON TABLE usuarios IS 'Almacena información de los usuarios registrados';
COMMENT ON COLUMN usuarios.username IS 'Nombre de usuario único para autenticación';
COMMENT ON COLUMN usuarios.email IS 'Correo electrónico único del usuario';
COMMENT ON COLUMN usuarios.creado_en IS 'Fecha y hora de registro del usuario';

--Lectura de comentario en tabla y columna
--Tablas
SELECT obj_description('usuarios'::regclass);

--Columnas
SELECT col_description('usuarios'::regclass, 2); --El 2 es la posicion de la col
SELECT col_description('usuarios'::regclass, 3);
SELECT col_description('usuarios'::regclass, 4);
```

### 1.2 Tabla de Perfiles de Voz

```sql
-- Crear tabla perfiles_voz
CREATE TABLE perfiles_voz (
    id SERIAL PRIMARY KEY,
    usuario_id INTEGER NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    palabra_magica TEXT NOT NULL DEFAULT 'aletza',
    vector_voz JSONB,
    muestras_tomadas INTEGER DEFAULT 0,
    estado TEXT DEFAULT 'incompleto',
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Agregar restricciones
ALTER TABLE perfiles_voz 
ADD CONSTRAINT estado_valido 
CHECK (estado IN ('incompleto', 'completo'));

ALTER TABLE perfiles_voz 
ADD CONSTRAINT muestras_validas 
CHECK (muestras_tomadas BETWEEN 0 AND 10);

-- Comentarios
COMMENT ON TABLE perfiles_voz IS 'Perfiles biométricos de voz para autenticación';
COMMENT ON COLUMN perfiles_voz.vector_voz IS 'Vector de embeddings de la voz del usuario';
COMMENT ON COLUMN perfiles_voz.muestras_tomadas IS 'Cantidad de muestras de voz registradas (máx 10)';
```

### 1.3 Tabla de Sesiones

```sql
-- Crear tabla sesiones
CREATE TABLE sesiones (
    id SERIAL PRIMARY KEY,
    usuario_id INTEGER NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    token TEXT UNIQUE NOT NULL,
    canal TEXT,
    iniciada_en TIMESTAMPTZ DEFAULT NOW(),
    ultima_actividad TIMESTAMPTZ DEFAULT NOW(),
    expira_en TIMESTAMPTZ DEFAULT NOW() + INTERVAL '1 hour'
);

-- Agregar restricción de canal
ALTER TABLE sesiones 
ADD CONSTRAINT canal_valido 
CHECK (canal IN ('telegram', 'web', 'app'));

-- Comentarios
COMMENT ON TABLE sesiones IS 'Registro de sesiones autenticadas de usuarios';
COMMENT ON COLUMN sesiones.token IS 'Token único de autenticación para la sesión';
COMMENT ON COLUMN sesiones.expira_en IS 'Fecha y hora de expiración de la sesión (1 hora después de iniciada)';
```

### 1.4 Tabla de Mensajes

```sql
-- Crear tabla mensajes
CREATE TABLE mensajes (
    id SERIAL PRIMARY KEY,
    sesion_id INTEGER NOT NULL REFERENCES sesiones(id) ON DELETE CASCADE,
    remitente_id INTEGER NOT NULL REFERENCES usuarios(id),
    contenido TEXT,
    modalidad TEXT,
    metadata_ia JSONB,
    enviado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Agregar restricción de modalidad
ALTER TABLE mensajes 
ADD CONSTRAINT modalidad_valida 
CHECK (modalidad IN ('texto', 'audio', 'imagen', 'video'));

-- Agregar restricción de contenido no vacío
ALTER TABLE mensajes 
ADD CONSTRAINT contenido_no_vacio 
CHECK (contenido IS NOT NULL OR metadata_ia IS NOT NULL);

-- Comentarios
COMMENT ON TABLE mensajes IS 'Mensajes enviados por usuarios al asistente';
COMMENT ON COLUMN mensajes.modalidad IS 'Tipo de mensaje: texto, audio, imagen, video';
COMMENT ON COLUMN mensajes.metadata_ia IS 'Resultados de procesamiento de IA en formato JSON';
```

### 1.5 Tabla de Logs de Ejecución

```sql
-- Crear tabla logs_ejecucion
CREATE TABLE logs_ejecucion (
    id SERIAL PRIMARY KEY,
    mensaje_id INTEGER NOT NULL REFERENCES mensajes(id) ON DELETE CASCADE,
    modulo TEXT NOT NULL,
    tiempo_ejecucion_ms INTEGER,
    exito BOOLEAN DEFAULT TRUE,
    error TEXT,
    ejecutado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Agregar restricción de tiempo positivo
ALTER TABLE logs_ejecucion 
ADD CONSTRAINT tiempo_positivo 
CHECK (tiempo_ejecucion_ms IS NULL OR tiempo_ejecucion_ms >= 0);

-- Comentarios
COMMENT ON TABLE logs_ejecucion IS 'Logs de ejecución de módulos IA';
COMMENT ON COLUMN logs_ejecucion.modulo IS 'Nombre del módulo de IA ejecutado';
COMMENT ON COLUMN logs_ejecucion.tiempo_ejecucion_ms IS 'Tiempo de ejecución en milisegundos';
```

---

## 🔧 Paso 2: Modificaciones con ALTER TABLE

### 2.1 Agregar Columnas

```sql
-- Agregar columna de país a usuarios
ALTER TABLE usuarios 
ADD COLUMN pais TEXT;

-- Agregar columna de ciudad a usuarios
ALTER TABLE usuarios 
ADD COLUMN ciudad TEXT;

-- Agregar columna de versión del asistente a sesiones
ALTER TABLE sesiones 
ADD COLUMN version_asistente TEXT DEFAULT '1.0';

-- Agregar columna de prioridad a logs
ALTER TABLE logs_ejecucion 
ADD COLUMN prioridad INTEGER DEFAULT 1;
```

### 2.2 Modificar Tipos de Datos

```sql
-- Cambiar username a longitud máxima 50
ALTER TABLE usuarios 
ALTER COLUMN username TYPE VARCHAR(50);

-- Cambiar email a longitud máxima 100
ALTER TABLE usuarios 
ALTER COLUMN email TYPE VARCHAR(100);

-- Ampliar longitud de token
ALTER TABLE sesiones 
ALTER COLUMN token TYPE VARCHAR(100);
```

### 2.3 Agregar Restricciones

```sql
-- Agregar restricción de formato para email
ALTER TABLE usuarios 
ADD CONSTRAINT email_valido 
CHECK (email LIKE '%@%');

-- Agregar restricción de unicidad compuesta
ALTER TABLE perfiles_voz 
ADD CONSTRAINT usuario_unico_perfil 
UNIQUE (usuario_id);

-- Agregar restricción de formato para token
ALTER TABLE sesiones 
ADD CONSTRAINT token_formato 
CHECK (token ~ '^[a-zA-Z0-9_]+$');
```

### 2.4 Agregar NOT NULL

```sql
-- Hacer obligatorio el campo nombre (si existiera)
-- Nota: En nuestras tablas no tenemos nombre, pero ejemplo:
-- ALTER TABLE usuarios ALTER COLUMN nombre SET NOT NULL;

-- Hacer obligatorio el campo canal
ALTER TABLE sesiones 
ALTER COLUMN canal SET NOT NULL;
```

### 2.5 Eliminar Columnas

```sql
-- Eliminar columna de prioridad
ALTER TABLE logs_ejecucion 
DROP COLUMN IF EXISTS prioridad;

-- Eliminar columna de versión del asistente
ALTER TABLE sesiones 
DROP COLUMN IF EXISTS version_asistente;
```

---

## 📊 Paso 3: Inserción de Datos Dummy

### 3.1 Insertar Usuarios

```sql
INSERT INTO usuarios (username, email, creado_en, pais, ciudad) VALUES
('carlos_dev', 'carlos.mendoza@email.com', '2024-01-15 10:30:00', 'Perú', 'Lima'),
('maria_lopez', 'maria.lopez@email.com', '2024-01-15 14:00:00', 'México', 'CDMX'),
('juan_torres', 'juan.torres@email.com', '2024-01-16 09:15:00', 'Argentina', 'Buenos Aires'),
('ana_rodriguez', 'ana.rodriguez@email.com', '2024-01-16 18:20:00', 'Colombia', 'Bogotá'),
('luis_fernandez', 'luis.fernandez@email.com', '2024-01-17 08:00:00', 'España', 'Madrid'),
('sofia_ramirez', 'sofia.ramirez@email.com', '2024-01-17 20:00:00', 'Chile', 'Santiago'),
('pedro_gomez', 'pedro.gomez@email.com', '2024-01-18 11:00:00', 'Argentina', 'Córdoba'),
('laura_martinez', 'laura.martinez@email.com', '2024-01-18 15:30:00', 'Colombia', 'Medellín'),
('diego_silva', 'diego.silva@email.com', '2024-01-19 07:30:00', 'Perú', 'Arequipa'),
('carmen_vega', 'carmen.vega@email.com', '2024-01-19 22:00:00', 'México', 'Guadalajara');

-- Verificar inserción
SELECT * FROM usuarios;
```

### 3.2 Insertar Perfiles de Voz

```sql
INSERT INTO perfiles_voz (usuario_id, palabra_magica, vector_voz, muestras_tomadas, estado) VALUES
(1, 'aletza', '{"embedding": [0.12, 0.45, 0.78, 0.23], "modelo": "v1"}', 10, 'completo'),
(2, 'aletza', '{"embedding": [0.34, 0.56, 0.12, 0.89], "modelo": "v1"}', 10, 'completo'),
(3, 'aletza', '{"embedding": [0.56, 0.78, 0.34, 0.12], "modelo": "v1"}', 8, 'incompleto'),
(4, 'aletza', '{"embedding": [0.78, 0.12, 0.56, 0.34], "modelo": "v1"}', 10, 'completo'),
(5, 'aletza', '{"embedding": [0.23, 0.67, 0.89, 0.45], "modelo": "v1"}', 5, 'incompleto'),
(6, 'aletza', '{"embedding": [0.45, 0.89, 0.23, 0.67], "modelo": "v1"}', 10, 'completo'),
(7, 'aletza', '{"embedding": [0.67, 0.34, 0.45, 0.78], "modelo": "v1"}', 10, 'completo'),
(8, 'aletza', '{"embedding": [0.89, 0.23, 0.67, 0.56], "modelo": "v1"}', 3, 'incompleto'),
(9, 'aletza', '{"embedding": [0.34, 0.78, 0.12, 0.45], "modelo": "v1"}', 10, 'completo'),
(10, 'aletza', '{"embedding": [0.56, 0.12, 0.78, 0.34], "modelo": "v1"}', 10, 'completo');

-- Verificar inserción
SELECT u.username, p.estado, p.muestras_tomadas 
FROM usuarios u
INNER JOIN perfiles_voz p ON u.id = p.usuario_id;
```

### 3.3 Insertar Sesiones

```sql
INSERT INTO sesiones (usuario_id, token, canal, iniciada_en, ultima_actividad, expira_en) VALUES
(1, 'tok_abc123', 'telegram', '2024-01-15 10:30:00', '2024-01-15 11:00:00', '2024-01-15 11:30:00'),
(1, 'tok_def456', 'web', '2024-01-16 09:15:00', '2024-01-16 09:45:00', '2024-01-16 10:15:00'),
(2, 'tok_ghi789', 'telegram', '2024-01-15 14:00:00', '2024-01-15 14:30:00', '2024-01-15 15:00:00'),
(2, 'tok_jkl012', 'app', '2024-01-16 18:20:00', '2024-01-16 18:50:00', '2024-01-16 19:20:00'),
(3, 'tok_mno345', 'web', '2024-01-15 08:00:00', '2024-01-15 08:30:00', '2024-01-15 09:00:00'),
(4, 'tok_pqr678', 'telegram', '2024-01-16 20:00:00', '2024-01-16 20:30:00', '2024-01-16 21:00:00'),
(5, 'tok_stu901', 'app', '2024-01-15 12:00:00', '2024-01-15 12:30:00', '2024-01-15 13:00:00'),
(6, 'tok_vwx234', 'telegram', '2024-01-16 11:00:00', '2024-01-16 11:30:00', '2024-01-16 12:00:00'),
(7, 'tok_yz5678', 'web', '2024-01-15 16:00:00', '2024-01-15 16:30:00', '2024-01-15 17:00:00'),
(8, 'tok_abc890', 'app', '2024-01-16 07:30:00', '2024-01-16 08:00:00', '2024-01-16 08:30:00'),
(9, 'tok_def123', 'telegram', '2024-01-15 22:00:00', '2024-01-15 22:30:00', '2024-01-15 23:00:00'),
(10, 'tok_ghi456', 'web', '2024-01-16 15:30:00', '2024-01-16 16:00:00', '2024-01-16 16:30:00');

-- Verificar inserción
SELECT u.username, s.canal, s.iniciada_en 
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id;
```

### 3.4 Insertar Mensajes

```sql
INSERT INTO mensajes (sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en) VALUES
-- Sesión 1 (usuario 1)
(1, 1, 'Hola Aletza, ¿cómo estás?', 'texto', '{"intencion": "saludo", "sentimiento": "positivo"}', '2024-01-15 10:30:15'),
(1, 1, 'Quiero saber el clima en Lima', 'texto', '{"intencion": "clima", "sentimiento": "neutral"}', '2024-01-15 10:31:00'),
(1, 1, 'Gracias por la información', 'texto', '{"intencion": "agradecimiento", "sentimiento": "positivo"}', '2024-01-15 10:32:00'),

-- Sesión 2 (usuario 1)
(2, 1, '¿Puedes ayudarme con soporte técnico?', 'texto', '{"intencion": "soporte", "sentimiento": "neutral"}', '2024-01-16 09:16:00'),
(2, 1, 'Mi cuenta no carga correctamente', 'audio', '{"intencion": "soporte", "sentimiento": "negativo", "transcripcion": "mi cuenta no carga correctamente"}', '2024-01-16 09:18:00'),

-- Sesión 3 (usuario 2)
(3, 2, 'aletza', 'audio', '{"intencion": "autenticacion", "sentimiento": "neutral"}', '2024-01-15 14:00:10'),
(3, 2, 'Reproduce mi playlist favorita', 'texto', '{"intencion": "musica", "sentimiento": "positivo"}', '2024-01-15 14:01:00'),
(3, 2, '¿Qué canciones tiene?', 'texto', '{"intencion": "consulta", "sentimiento": "neutral"}', '2024-01-15 14:02:30'),

-- Sesión 4 (usuario 2)
(4, 2, 'Dame recomendaciones de películas', 'texto', '{"intencion": "recomendacion", "sentimiento": "neutral"}', '2024-01-16 18:21:00'),

-- Sesión 5 (usuario 3)
(5, 3, '¿Cuál es el horario de atención?', 'texto', '{"intencion": "informacion", "sentimiento": "neutral"}', '2024-01-15 08:01:00'),

-- Sesión 6 (usuario 4)
(6, 4, 'Hola', 'texto', '{"intencion": "saludo", "sentimiento": "neutral"}', '2024-01-16 20:01:00'),
(6, 4, 'Quiero aprender SQL', 'texto', '{"intencion": "educacion", "sentimiento": "positivo"}', '2024-01-16 20:05:00'),

-- Sesión 7 (usuario 5)
(7, 5, '¿Cómo configuro mi cuenta?', 'texto', '{"intencion": "soporte", "sentimiento": "neutral"}', '2024-01-15 12:05:00'),

-- Sesión 8 (usuario 6)
(8, 6, '¡Excelente servicio!', 'texto', '{"intencion": "feedback", "sentimiento": "positivo"}', '2024-01-16 11:05:00'),

-- Sesión 9 (usuario 7)
(9, 7, 'Necesito cambiar mi contraseña', 'texto', '{"intencion": "soporte", "sentimiento": "neutral"}', '2024-01-15 16:02:00'),

-- Sesión 10 (usuario 8)
(10, 8, 'Buenos días', 'texto', '{"intencion": "saludo", "sentimiento": "positivo"}', '2024-01-16 07:32:00'),

-- Sesión 11 (usuario 9)
(11, 9, 'Crear recordatorio para mañana', 'texto', '{"intencion": "recordatorio", "sentimiento": "neutral"}', '2024-01-15 22:05:00'),

-- Sesión 12 (usuario 10)
(12, 10, '¿Cómo está el clima en Barcelona?', 'texto', '{"intencion": "clima", "sentimiento": "neutral"}', '2024-01-16 15:32:00');

-- Verificar inserción
SELECT COUNT(*) AS total_mensajes FROM mensajes;
```

### 3.5 Insertar Logs de Ejecución

```sql
INSERT INTO logs_ejecucion (mensaje_id, modulo, tiempo_ejecucion_ms, exito, error, ejecutado_en) VALUES
-- Mensajes de usuario 1
(1, 'clasificacion_nlp', 45, TRUE, NULL, '2024-01-15 10:30:16'),
(1, 'generacion_respuesta', 78, TRUE, NULL, '2024-01-15 10:30:16'),
(2, 'clasificacion_nlp', 52, TRUE, NULL, '2024-01-15 10:31:01'),
(2, 'api_clima', 234, TRUE, NULL, '2024-01-15 10:31:02'),
(2, 'generacion_respuesta', 67, TRUE, NULL, '2024-01-15 10:31:02'),
(3, 'clasificacion_nlp', 38, TRUE, NULL, '2024-01-15 10:32:01'),
(3, 'generacion_respuesta', 56, TRUE, NULL, '2024-01-15 10:32:01'),
(4, 'clasificacion_nlp', 48, TRUE, NULL, '2024-01-16 09:16:01'),
(4, 'generacion_respuesta', 72, TRUE, NULL, '2024-01-16 09:16:01'),
(5, 'transcripcion_audio', 156, TRUE, NULL, '2024-01-16 09:18:05'),
(5, 'clasificacion_nlp', 55, TRUE, NULL, '2024-01-16 09:18:05'),

-- Mensajes de usuario 2
(6, 'autenticacion_biometrica', 189, TRUE, NULL, '2024-01-15 14:00:11'),
(7, 'clasificacion_nlp', 42, TRUE, NULL, '2024-01-15 14:01:01'),
(7, 'api_musica', 312, TRUE, NULL, '2024-01-15 14:01:02'),
(8, 'clasificacion_nlp', 44, TRUE, NULL, '2024-01-15 14:02:31'),
(8, 'generacion_respuesta', 61, TRUE, NULL, '2024-01-15 14:02:31'),
(9, 'clasificacion_nlp', 41, TRUE, NULL, '2024-01-16 18:21:01'),
(9, 'api_recomendaciones', 287, TRUE, NULL, '2024-01-16 18:21:02'),

-- Mensajes con errores simulados
(10, 'clasificacion_nlp', 89, FALSE, 'Timeout en API externa', '2024-01-15 08:01:01'),
(11, 'clasificacion_nlp', 37, TRUE, NULL, '2024-01-16 20:01:01'),
(12, 'clasificacion_nlp', 42, TRUE, NULL, '2024-01-16 20:05:01'),
(13, 'clasificacion_nlp', 51, TRUE, NULL, '2024-01-15 12:05:01'),
(14, 'clasificacion_nlp', 46, TRUE, NULL, '2024-01-16 11:05:01'),
(15, 'clasificacion_nlp', 44, TRUE, NULL, '2024-01-15 16:02:01'),
(16, 'clasificacion_nlp', 43, TRUE, NULL, '2024-01-16 07:32:01'),
(17, 'api_recordatorios', 178, TRUE, NULL, '2024-01-15 22:05:01'),
(18, 'clasificacion_nlp', 49, TRUE, NULL, '2024-01-16 15:32:01'),
(18, 'api_clima', 221, TRUE, NULL, '2024-01-16 15:32:02');

-- Verificar inserción
SELECT modulo, COUNT(*) AS total, AVG(tiempo_ejecucion_ms) AS tiempo_promedio
FROM logs_ejecucion
GROUP BY modulo
ORDER BY total DESC;
```

---

## 🗑️ Paso 4: DROP y TRUNCATE (Ejemplos)

### 4.1 TRUNCATE (Vaciar Tablas)

```sql
-- Vaciar tabla de logs (mantiene estructura)
TRUNCATE TABLE logs_ejecucion;

-- Vaciar múltiples tablas
TRUNCATE TABLE logs_ejecucion, mensajes;

-- TRUNCATE con reinicio de secuencias
TRUNCATE TABLE logs_ejecucion RESTART IDENTITY;
```

### 4.2 DROP (Eliminar Objetos)

```sql
-- Eliminar restricción
ALTER TABLE usuarios DROP CONSTRAINT IF EXISTS email_valido;

-- Eliminar columna
ALTER TABLE logs_ejecucion DROP COLUMN IF EXISTS prioridad;

-- Eliminar tabla (¡cuidado!)
DROP TABLE IF EXISTS logs_backup;

-- NOTA: En Supabase no se puede DROP DATABASE
```


Finalmente, para dejar todo desde el inicio:

```sql
/*
-- =========================
-- ELIMINAR TABLAS EXISTENTES
-- =========================

DROP TABLE IF EXISTS logs_ejecucion CASCADE;
DROP TABLE IF EXISTS mensajes CASCADE;
DROP TABLE IF EXISTS sesiones CASCADE;
DROP TABLE IF EXISTS perfiles_voz CASCADE;
DROP TABLE IF EXISTS usuarios CASCADE;

*/
```

o de forma total incluyendo objetos, indices, etc:

```sql
/*
-- ELIMINAR TODO DE UNA VEZ
DROP SCHEMA public CASCADE;
CREATE SCHEMA public;

-- Restaurar permisos básicos
GRANT ALL ON SCHEMA public TO postgres;
GRANT ALL ON SCHEMA public TO public;
*/
```