# 📘 Laboratorio 2: DML (Data Manipulation Language) - Caso Aletza

## 🎯 Objetivo
Aplicar los conceptos de DML (SELECT, INSERT, UPDATE, DELETE, MERGE) al caso del asistente virtual Aletza, utilizando los datos insertados en el Laboratorio 1.

---

## 📋 Requisitos Previos
- Tener ejecutado el Laboratorio 1 con todas las tablas creadas y datos insertados
- Conectado a Supabase con el SQL Editor abierto

---

## 🔍 Paso 1: SELECT - Consultas Básicas

### 1.1 SELECT con todas las columnas

```sql
-- Seleccionar todos los usuarios
SELECT * FROM usuarios;

-- Seleccionar todos los perfiles de voz
SELECT * FROM perfiles_voz;

-- Seleccionar todas las sesiones
SELECT * FROM sesiones;

-- Seleccionar todos los mensajes
SELECT * FROM mensajes;

-- Seleccionar todos los logs
SELECT * FROM logs_ejecucion;
```

### 1.2 SELECT con columnas específicas

```sql
-- Mostrar solo username y email de usuarios
SELECT username, email FROM usuarios;

-- Mostrar usuario_id, estado y muestras_tomadas de perfiles de voz
SELECT usuario_id, estado, muestras_tomadas FROM perfiles_voz;

-- Mostrar token, canal y expira_en de sesiones
SELECT token, canal, expira_en FROM sesiones;

-- Mostrar contenido y modalidad de mensajes
SELECT contenido, modalidad FROM mensajes;

-- Mostrar modulo, tiempo_ejecucion_ms y exito de logs
SELECT modulo, tiempo_ejecucion_ms, exito FROM logs_ejecucion;

-- Concatenación: "Token [canal] expira en: {expira_en}"
SELECT 
    'Token ' || token || ' [' || canal || '] expira en: ' || expira_en AS info_sesion
FROM sesiones;
```

### 1.3 SELECT con ALIAS (AS)

```sql
-- Usar alias para renombrar columnas
SELECT 
    username AS "Nombre de Usuario",
    email AS "Correo Electrónico",
    creado_en AS "Fecha Registro"
FROM usuarios;

-- Alias para tablas
SELECT u.username, u.email, p.estado
FROM usuarios AS u
INNER JOIN perfiles_voz AS p ON u.id = p.usuario_id;

-- El AS es opcional
SELECT u.username, u.email, p.estado
FROM usuarios u
INNER JOIN perfiles_voz p ON u.id = p.usuario_id;
```

### 1.4 SELECT con DISTINCT

```sql
-- Países únicos de usuarios
SELECT DISTINCT pais FROM usuarios WHERE pais IS NOT NULL;

-- Canales únicos usados en sesiones
SELECT DISTINCT canal FROM sesiones;

-- Modalidades únicas de mensajes
SELECT DISTINCT modalidad FROM mensajes;

-- Módulos únicos en logs
SELECT DISTINCT modulo FROM logs_ejecucion;

-- Combinaciones únicas de canal y usuario
SELECT DISTINCT canal, usuario_id FROM sesiones;
```

### 1.5 SELECT con LIMIT y OFFSET

```sql
-- Mostrar primeros 3 usuarios
SELECT * FROM usuarios LIMIT 3;

-- Mostrar 5 mensajes
SELECT * FROM mensajes LIMIT 5;

-- Paginación: saltar 2 y mostrar 3
SELECT * FROM usuarios ORDER BY id LIMIT 3 OFFSET 2;

-- Mostrar los 5 logs más recientes
SELECT * FROM logs_ejecucion 
ORDER BY ejecutado_en DESC 
LIMIT 5;
```

---

## 📝 Paso 2: SELECT con WHERE (Filtrado)

### 2.1 Operadores de Comparación

```sql
-- Usuarios de Perú
SELECT username, email, pais 
FROM usuarios 
WHERE pais = 'Perú';

-- Usuarios que NO son de Perú
SELECT username, email, pais 
FROM usuarios 
WHERE pais != 'Perú';

-- Mensajes con ID mayor a 10
SELECT id, contenido FROM mensajes WHERE id > 10;

-- Mensajes con ID menor o igual a 5
SELECT id, contenido FROM mensajes WHERE id <= 5;

-- Sesiones que expiran después del 2024-01-16
SELECT id, usuario_id, expira_en 
FROM sesiones 
WHERE expira_en > '2024-01-16';
```

### 2.2 BETWEEN (Rangos)

```sql
-- Usuarios con ID entre 3 y 7
SELECT id, username FROM usuarios WHERE id BETWEEN 3 AND 7;

-- Mensajes enviados entre 2024-01-15 y 2024-01-16
SELECT id, contenido, enviado_en 
FROM mensajes 
WHERE enviado_en BETWEEN '2024-01-15' AND '2024-01-16';

-- Sesiones con expiración en enero 2024
SELECT id, usuario_id, expira_en 
FROM sesiones 
WHERE expira_en BETWEEN '2024-01-01' AND '2024-01-31';
```

### 2.3 IN (Múltiples valores)

```sql
-- Usuarios de Perú, México y Argentina
SELECT username, pais 
FROM usuarios 
WHERE pais IN ('Perú', 'México', 'Argentina');

-- Sesiones en canales específicos
SELECT id, usuario_id, canal 
FROM sesiones 
WHERE canal IN ('telegram', 'web');

-- Mensajes de modalidad audio o video
SELECT id, contenido, modalidad 
FROM mensajes 
WHERE modalidad IN ('audio', 'video');

-- NOT IN (excluir valores)
SELECT username, pais 
FROM usuarios 
WHERE pais NOT IN ('Perú', 'México');
```

### 2.4 LIKE (Búsqueda de patrones)

```sql
-- Usuarios con email que termina en .com
SELECT username, email FROM usuarios WHERE email LIKE '%.com';

-- Usuarios con email de Gmail
SELECT username, email FROM usuarios WHERE email LIKE '%@gmail.com';

-- Usuarios cuyo username empieza con 'c'
SELECT username FROM usuarios WHERE username LIKE 'c%';

-- Usuarios cuyo username termina con 'z'
SELECT username FROM usuarios WHERE username LIKE '%z';

-- Usuarios cuyo username contiene 'ar'
SELECT username FROM usuarios WHERE username LIKE '%ar%';

-- Mensajes que contienen 'clima' (insensible a mayúsculas)
SELECT contenido FROM mensajes WHERE contenido ILIKE '%clima%';
```

### 2.5 IS NULL / IS NOT NULL

```sql
-- Usuarios con país registrado
SELECT username, pais FROM usuarios WHERE pais IS NOT NULL;

-- Usuarios sin país
SELECT username, pais FROM usuarios WHERE pais IS NULL;

-- Logs sin error
SELECT id, modulo, error FROM logs_ejecucion WHERE error IS NULL;

-- Logs con error
SELECT id, modulo, error FROM logs_ejecucion WHERE error IS NOT NULL;
```

### 2.6 Operadores Lógicos (AND, OR, NOT)

```sql
-- Usuarios de Perú que se registraron después del 2024-01-17
SELECT username, pais, creado_en 
FROM usuarios 
WHERE pais = 'Perú' AND creado_en > '2024-01-17';

-- Usuarios de Perú O México
SELECT username, pais 
FROM usuarios 
WHERE pais = 'Perú' OR pais = 'México';

-- Mensajes que NO son de texto
SELECT id, contenido, modalidad 
FROM mensajes 
WHERE modalidad != 'texto';

-- Combinación compleja
SELECT * FROM logs_ejecucion 
WHERE (modulo = 'clasificacion_nlp' OR modulo = 'api_clima')
AND exito = TRUE
AND tiempo_ejecucion_ms > 50;
```

---

## 📊 Paso 3: SELECT con ORDER BY

### 3.1 Ordenamiento Básico

```sql
-- Usuarios ordenados por username ascendente
SELECT username, email FROM usuarios ORDER BY username;

-- Usuarios ordenados por email descendente
SELECT username, email FROM usuarios ORDER BY email DESC;

-- Mensajes ordenados por fecha de envío (más recientes primero)
SELECT id, contenido, enviado_en 
FROM mensajes 
ORDER BY enviado_en DESC;
```

### 3.2 Ordenamiento Múltiple

```sql
-- Usuarios ordenados por país y luego por username
SELECT username, pais FROM usuarios 
ORDER BY pais ASC, username ASC;

-- Logs ordenados por módulo y luego por tiempo de ejecución
SELECT modulo, tiempo_ejecucion_ms, exito 
FROM logs_ejecucion 
ORDER BY modulo, tiempo_ejecucion_ms DESC;

-- Mensajes ordenados por remitente y fecha
SELECT remitente_id, contenido, enviado_en 
FROM mensajes 
ORDER BY remitente_id, enviado_en DESC;
```

### 3.3 Ordenamiento con NULLS

```sql
-- Pais con NULL al final
SELECT username, pais FROM usuarios 
ORDER BY pais NULLS LAST;

-- País con NULL al principio
SELECT username, pais FROM usuarios 
ORDER BY pais NULLS FIRST;
```

---

## ✏️ Paso 4: INSERT - Inserción de Datos




### 4.3 INSERT desde SELECT

```sql
-- Crear tabla de respaldo (primero la creamos si no existe)
CREATE TABLE IF NOT EXISTS usuarios_backup AS 
SELECT * FROM usuarios WHERE 1=0;  -- Solo estructura

-- Insertar usuarios de Perú en tabla de respaldo
INSERT INTO usuarios_backup (id, username, email, creado_en, pais, ciudad)
SELECT id, username, email, creado_en, pais, ciudad
FROM usuarios 
WHERE pais = 'Perú';

-- Verificar
SELECT * FROM usuarios_backup;
```

---

## 🔄 Paso 5: UPDATE - Actualización de Datos

### 5.1 UPDATE Básico

```sql
-- Actualizar país de un usuario
UPDATE usuarios 
SET pais = 'España' 
WHERE username = 'carlos_dev';

-- Verificar cambio
SELECT username, pais FROM usuarios WHERE username = 'carlos_dev';

-- Actualizar estado de perfil de voz
UPDATE perfiles_voz 
SET estado = 'completo', muestras_tomadas = 10 
WHERE usuario_id = 3;

-- Verificar
SELECT usuario_id, estado, muestras_tomadas 
FROM perfiles_voz 
WHERE usuario_id = 3;
```

### 5.2 UPDATE Múltiples Columnas

```sql
-- Actualizar varias columnas a la vez
UPDATE usuarios 
SET pais = 'México', ciudad = 'Cancún' 
WHERE username = 'maria_lopez';

-- Verificar
SELECT username, pais, ciudad 
FROM usuarios 
WHERE username = 'maria_lopez';
```

### 5.3 UPDATE con Expresiones

```sql
-- Incrementar muestras_tomadas en 1 para perfiles incompletos
UPDATE perfiles_voz 
SET muestras_tomadas = muestras_tomadas + 1 
WHERE estado = 'incompleto';

-- Verificar
SELECT usuario_id, muestras_tomadas, estado 
FROM perfiles_voz 
WHERE estado = 'incompleto';

-- Actualizar timestamp de última actividad
UPDATE sesiones 
SET ultima_actividad = NOW() 
WHERE usuario_id = 1;
```





---

## 🗑️ Paso 6: DELETE - Eliminación de Datos

### 6.1 DELETE Básico

```sql
-- Eliminar usuario test
DELETE FROM usuarios WHERE username = 'test1';

-- Verificar
SELECT * FROM usuarios WHERE username LIKE 'test%';

-- Eliminar sesiones de un usuario específico
DELETE FROM sesiones WHERE usuario_id = 11;

-- Verificar
SELECT * FROM sesiones WHERE usuario_id = 11;
```

### 6.2 DELETE con Condiciones

```sql
-- Eliminar logs con error
DELETE FROM logs_ejecucion WHERE exito = FALSE;

-- Verificar
SELECT * FROM logs_ejecucion WHERE exito = FALSE;

-- Eliminar mensajes antiguos (antes de enero 2024)
DELETE FROM mensajes WHERE enviado_en < '2024-01-01';
```



### 6.5 CUIDADO: DELETE sin WHERE

```sql
-- ¡PELIGRO! Esto elimina todos los registros
-- DELETE FROM logs_ejecucion;

-- Mejor usar TRUNCATE si se quiere vaciar toda la tabla
TRUNCATE TABLE logs_ejecucion;
```

