---
layout: default
---

# SQL - PARTE 1

En las sesiones anteriores hemos aprendido a modelar bases de datos, normalizar y definir estructuras. Ahora es el momento de interactuar con los datos: insertarlos, consultarlos, modificarlos y eliminarlos. El lenguaje SQL (Structured Query Language) es el estándar para estas tareas. En esta sesión nos centraremos en el sublenguaje DML (Data Manipulation Language) y en las operaciones de conjuntos que nos permiten combinar información de múltiples tablas: los JOINs. También exploraremos cómo el motor de base de datos interpreta y ejecuta nuestras consultas, qué sucede a nivel hardware, y cómo escribir SQL eficiente. 

## El Lenguaje SQL y sus Sublenguajes

SQL se divide en varios sublenguajes según su función:

- **DDL (Data Definition Language):** CREATE, ALTER, DROP -- definen la estructura.

- **DML (Data Manipulation Language):** SELECT, INSERT, UPDATE, DELETE -- manipulan los datos.

- **DCL (Data Control Language):** GRANT, REVOKE -- controlan permisos.

- **TCL (Transaction Control Language):** COMMIT, ROLLBACK -- gestionan transacciones.

En esta sesión profundizaremos en DML, con especial énfasis en SELECT y JOINs.

# Lenguaje SQL: Sublenguajes y Operaciones Fundamentales

## DDL (Data Definition Language)

El sublenguaje de definición de datos se utiliza para crear, modificar y eliminar la estructura de los objetos de la base de datos (tablas, índices, vistas, etc.). Los comandos DDL suelen ser auto-commit, es decir, no se pueden deshacer con ROLLBACK en la mayoría de los motores.

### CREATE

Crea nuevos objetos. El más común es `CREATE TABLE`.

```sql
CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    ciudad VARCHAR(50),
    fecha_registro DATE DEFAULT CURRENT_DATE
);
```


### ALTER

Modifica la estructura de un objeto existente.

```sql
-- Añadir una columna
ALTER TABLE clientes ADD COLUMN telefono VARCHAR(20);

-- Modificar tipo de dato
ALTER TABLE clientes ALTER COLUMN nombre TYPE VARCHAR(200);

-- Añadir una restricción
ALTER TABLE clientes ADD CONSTRAINT email_unico UNIQUE (email);
```

### DROP

Elimina objetos de la base de datos. ¡Cuidado! Es irreversible (a menos que tengas backup).

```sql
DROP TABLE clientes; -- Elimina la tabla y todos sus datos
DROP INDEX idx_ciudad;
DROP VIEW clientes_madrid;
```

### TRUNCATE

Elimina todas las filas de una tabla de forma rápida (no genera logs por fila, a diferencia de DELETE). Es DDL porque no se puede deshacer con ROLLBACK en muchos motores.

```sql
TRUNCATE TABLE clientes;
```

### Restricciones de Integridad en DDL

Las restricciones (constraints) se definen en DDL y garantizan la integridad de los datos:

- **PRIMARY KEY**: Identifica unívocamente cada fila.

- **FOREIGN KEY**: Mantiene integridad referencial.

- **UNIQUE**: Valores únicos en una columna o combinación.

- **CHECK**: Valida que los datos cumplan una condición (ej. salario \> 0).

- **NOT NULL**: La columna no puede tener valores nulos.

Ejemplo con todas ellas:

```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INT NOT NULL,
    fecha DATE NOT NULL,
    total DECIMAL(10,2) CHECK (total >= 0),
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);
```

## DML (Data Manipulation Language)

El sublenguaje de manipulación de datos permite consultar, insertar, modificar y eliminar los datos almacenados. Es el más utilizado en el día a día.

### SELECT

La sentencia SELECT recupera datos de una o más tablas. Ya la hemos visto en detalle, pero recordemos su sintaxis básica:

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY columna
LIMIT n;
```



### INSERT

Inserta nuevas filas en una tabla.

```sql
INSERT INTO clientes (nombre, email, ciudad) VALUES
('Carlos Ruiz', 'carlos@mail.com', 'Valencia');
```

También se puede insertar desde una consulta:

```sql
INSERT INTO clientes_vip (nombre, email)
SELECT nombre, email FROM clientes WHERE ciudad = 'Madrid';
```

### UPDATE

Modifica filas existentes.

```sql
UPDATE clientes SET ciudad = 'Sevilla' WHERE id = 5;
```

Siempre usar WHERE para no actualizar toda la tabla.

### DELETE

Elimina filas.

```sql
DELETE FROM clientes WHERE email IS NULL;
```

Sin WHERE, elimina todas las filas (pero la tabla sigue existiendo). Para vaciado masivo, TRUNCATE es más rápido.

### MERGE (UPSERT)

Combina insercion y actualizacion en una sola operacion. Algunos motores (SQL Server, Oracle, PostgreSQL con ON CONFLICT) permiten insertar o actualizar según exista o no la fila. Es muy útil en sincronización de datos.

```sql
-- PostgreSQL
INSERT INTO clientes (id, nombre, email)
VALUES (1, 'Ana', 'ana@mail.com')
ON CONFLICT (id) DO UPDATE SET nombre = EXCLUDED.nombre, email = EXCLUDED.email;
```

## DCL (Data Control Language)

El sublenguaje de control de datos gestiona los permisos y accesos a los objetos de la base de datos. Es fundamental en entornos multiusuario y para cumplir normativas de seguridad.

### GRANT

Otorga privilegios a usuarios o roles.

```sql
-- Otorgar permiso de SELECT sobre la tabla clientes al usuario 'analista'
GRANT SELECT ON clientes TO analista;

-- Otorgar todos los privilegios sobre la base de datos
GRANT ALL PRIVILEGES ON DATABASE mi_bd TO admin;
```

Los privilegios comunes son: SELECT, INSERT, UPDATE, DELETE, REFERENCES, TRIGGER, CREATE, CONNECT, etc.

### REVOKE

Revoca permisos previamente otorgados.

```sql
REVOKE DELETE ON clientes FROM analista;
```

## TCL (Transaction Control Language)

El sublenguaje de control de transacciones permite gestionar los cambios realizados con DML, agrupándolos en unidades lógicas que pueden confirmarse o deshacerse.

### BEGIN / START TRANSACTION

Inicia una transacción. En algunos motores (PostgreSQL) se usa BEGIN; en MySQL, START TRANSACTION.

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
-- Aún no son permanentes
```

### COMMIT

Confirma todos los cambios realizados en la transacción actual, haciéndolos permanentes.

```sql
COMMIT;
```

### ROLLBACK

Deshace todos los cambios realizados desde el BEGIN, volviendo al estado anterior.

```sql
ROLLBACK;
```

### SAVEPOINT

Permite establecer puntos intermedios dentro de una transacción para poder deshacer solo una parte.

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
SAVEPOINT despues_de_restar;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
-- Si algo falla, podemos retroceder al savepoint
ROLLBACK TO SAVEPOINT despues_de_restar;
-- Luego podemos continuar o hacer COMMIT
COMMIT;
```





---

## El Lenguaje SQL y sus Sublenguajes

SQL se divide en varios sublenguajes según su función. Conocer cada uno es esencial para entender el alcance del lenguaje:

| Sublenguaje | Comandos principales | Propósito |
|-------------|---------------------|-----------|
| **DDL** (Data Definition Language) | CREATE, ALTER, DROP, TRUNCATE | Definir y modificar la estructura de objetos |
| **DML** (Data Manipulation Language) | SELECT, INSERT, UPDATE, DELETE | Manipular los datos almacenados |
| **DCL** (Data Control Language) | GRANT, REVOKE | Controlar permisos y accesos |
| **TCL** (Transaction Control Language) | BEGIN, COMMIT, ROLLBACK | Gestionar transacciones |

En esta sesión nos enfocaremos principalmente en **DML**, con especial énfasis en la sentencia **SELECT** y las operaciones de **JOIN**.

---

## Introducción a PostgreSQL

### 1.2 Tipos de Datos en PostgreSQL

PostgreSQL ofrece una amplia variedad de tipos de datos. Conocerlos es fundamental para diseñar esquemas adecuados.

#### Tipos Numéricos

| Tipo | Descripción | Rango / Precisión | Ejemplo |
|------|-------------|-------------------|---------|
| `SMALLINT` | Entero pequeño | -32,768 a 32,767 | `edad SMALLINT` |
| `INTEGER` | Entero estándar | -2.1e9 a 2.1e9 | `id INTEGER` |
| `BIGINT` | Entero grande | -9.2e18 a 9.2e18 | `transacciones BIGINT` |
| `DECIMAL(p,s)` | Número decimal exacto | Precisión definida | `precio DECIMAL(10,2)` |
| `NUMERIC(p,s)` | Sinónimo de DECIMAL | Precisión definida | `saldo NUMERIC(12,2)` |
| `REAL` | Punto flotante (4 bytes) | 6 decimales | `temperatura REAL` |
| `DOUBLE PRECISION` | Punto flotante (8 bytes) | 15 decimales | `coordenada DOUBLE` |
| `SERIAL` | Auto-incremental | Entero | `id SERIAL PRIMARY KEY` |
| `BIGSERIAL` | Auto-incremental grande | Bigint | `id BIGSERIAL` |

```sql
-- Ejemplos de tipos numéricos
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    precio DECIMAL(10,2) NOT NULL,  -- hasta 99,999,999.99
    stock INTEGER DEFAULT 0,
    peso REAL,
    sku BIGINT UNIQUE
);
```

#### Tipos de Texto

| Tipo | Descripción | Longitud | Ejemplo |
|------|-------------|----------|---------|
| `CHAR(n)` | Longitud fija, rellena con espacios | fija | `codigo CHAR(10)` |
| `VARCHAR(n)` | Longitud variable con límite | hasta n | `nombre VARCHAR(100)` |
| `TEXT` | Longitud variable ilimitada | ilimitada | `descripcion TEXT` |

```sql
-- Ejemplos de tipos texto
CREATE TABLE articulos (
    titulo VARCHAR(200) NOT NULL,
    contenido TEXT,
    codigo_barras CHAR(13)  -- exactamente 13 caracteres
);
```

#### Tipos de Fecha y Hora

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| `DATE` | Solo fecha (año-mes-día) | `'2024-01-15'` |
| `TIME` | Solo hora | `'14:30:00'` |
| `TIMESTAMP` | Fecha y hora sin zona horaria | `'2024-01-15 14:30:00'` |
| `TIMESTAMPTZ` | Fecha y hora con zona horaria | `'2024-01-15 14:30:00-03'` |
| `INTERVAL` | Intervalo de tiempo | `INTERVAL '1 day'` |

```sql
-- Ejemplos de tipos fecha/hora
CREATE TABLE eventos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100),
    fecha_inicio DATE,
    hora_evento TIME,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    duracion INTERVAL
);
```

#### Tipos Booleanos

```sql
-- Tipo BOOLEAN: TRUE, FALSE, NULL
CREATE TABLE configuraciones (
    usuario_id INTEGER,
    activo BOOLEAN DEFAULT TRUE,
    notificaciones BOOLEAN NOT NULL DEFAULT FALSE
);
```

#### Tipos Especiales en PostgreSQL

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| `JSON` | Almacena JSON válido | `'{"clave": "valor"}'` |
| `JSONB` | JSON binario (más eficiente) | `'{"preferencias": {"tema": "oscuro"}}'` |
| `UUID` | Identificador único universal | `gen_random_uuid()` |
| `ARRAY` | Arreglo de elementos | `textos TEXT[]` |
| `ENUM` | Tipo enumerado | `CREATE TYPE estado AS ENUM ('activo', 'inactivo')` |

```sql
-- Ejemplos de tipos especiales
CREATE TABLE usuarios_avanzados (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    metadata JSONB,
    etiquetas TEXT[],
    preferencias JSONB DEFAULT '{"tema": "claro"}'::JSONB
);
```

### 1.3 Restricciones de Integridad (Constraints)

Las restricciones garantizan la calidad e integridad de los datos:

```sql
CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,                    -- PRIMARY KEY
    usuario_id INTEGER NOT NULL,              -- NOT NULL
    email VARCHAR(100) UNIQUE,                -- UNIQUE
    fecha DATE DEFAULT CURRENT_DATE,
    total DECIMAL(10,2) CHECK (total >= 0),   -- CHECK
    estado VARCHAR(20) DEFAULT 'pendiente',
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id)  -- FOREIGN KEY
);
```

---

## Módulo 2: Consultas Fundamentales (SELECT Básico)

### 2.1 Sintaxis Básica SELECT

La sentencia SELECT recupera datos de una o más tablas.

```sql
-- Seleccionar todas las columnas
SELECT * FROM usuarios;

-- Seleccionar columnas específicas
SELECT nombre, email FROM usuarios;

-- Seleccionar con condiciones
SELECT nombre, edad FROM usuarios WHERE edad >= 18;
```

### 2.2 Alias (AS) para Columnas y Tablas

Los alias permiten renombrar columnas o tablas en los resultados:

```sql
-- Alias para columna
SELECT nombre AS "Nombre Completo", email AS Correo FROM usuarios;

-- Alias para tabla (útil en JOINs)
SELECT u.nombre, u.email 
FROM usuarios AS u
WHERE u.activo = true;

-- El AS es opcional
SELECT u.nombre, u.email 
FROM usuarios u;
```

### 2.3 Operadores Aritméticos

```sql
-- Operadores básicos: +, -, *, /, % (módulo)
SELECT 
    nombre,
    precio,
    precio * 1.21 AS precio_con_iva,
    precio * 0.10 AS descuento,
    stock - vendidos AS stock_disponible
FROM productos;
```

### 2.4 Concatenación de Strings (||)

```sql
-- Concatenar con ||
SELECT nombre || ' ' || apellido AS nombre_completo 
FROM usuarios;

-- Concatenar con texto fijo
SELECT 'Usuario: ' || nombre || ' (ID: ' || id || ')' AS descripcion 
FROM usuarios;
```

### 2.5 DISTINCT para Eliminar Duplicados

```sql
-- Obtener valores únicos
SELECT DISTINCT ciudad FROM usuarios;

-- Combinaciones únicas
SELECT DISTINCT ciudad, pais FROM usuarios;

-- Contar valores distintos
SELECT COUNT(DISTINCT ciudad) FROM usuarios;
```

### 2.6 LIMIT y OFFSET para Paginación

```sql
-- Limitar resultados
SELECT * FROM usuarios LIMIT 10;

-- Paginación: saltar 20 registros y mostrar 10
SELECT * FROM usuarios LIMIT 10 OFFSET 20;

-- Combinar con ORDER BY para paginación consistente
SELECT * FROM usuarios 
ORDER BY id 
LIMIT 10 OFFSET 20;
```

---

## Módulo 3: Filtrado de Datos (WHERE)

### 3.1 Operadores de Comparación

| Operador | Significado |
|----------|-------------|
| `=` | Igual |
| `<>` o `!=` | Diferente |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual |
| `<=` | Menor o igual |

```sql
-- Ejemplos
SELECT * FROM usuarios WHERE edad = 25;
SELECT * FROM productos WHERE precio <> 0;
SELECT * FROM pedidos WHERE total > 1000;
SELECT * FROM empleados WHERE salario >= 5000;
```

### 3.2 BETWEEN para Rangos

```sql
-- Edad entre 18 y 30 años (inclusive)
SELECT * FROM usuarios 
WHERE edad BETWEEN 18 AND 30;

-- Fechas en un rango
SELECT * FROM pedidos 
WHERE fecha BETWEEN '2024-01-01' AND '2024-12-31';

-- Equivalente sin BETWEEN
SELECT * FROM usuarios 
WHERE edad >= 18 AND edad <= 30;
```

### 3.3 IN para Múltiples Valores

```sql
-- Ciudades específicas
SELECT * FROM usuarios 
WHERE ciudad IN ('Madrid', 'Barcelona', 'Valencia');

-- IDs específicos
SELECT * FROM pedidos 
WHERE estado IN ('completado', 'enviado');

-- Equivalente con OR
SELECT * FROM usuarios 
WHERE ciudad = 'Madrid' OR ciudad = 'Barcelona' OR ciudad = 'Valencia';

-- NOT IN (excluir valores)
SELECT * FROM usuarios 
WHERE ciudad NOT IN ('Madrid', 'Barcelona');
```

### 3.4 LIKE y Patrones (%, _)

`LIKE` permite búsquedas con patrones:

| Comodín | Significado | Ejemplo |
|---------|-------------|---------|
| `%` | Cualquier secuencia de caracteres | `'%gmail%'` contiene "gmail" |
| `_` | Un solo carácter | `'___'` exactamente 3 caracteres |

```sql
-- Empieza con 'A'
SELECT * FROM usuarios WHERE nombre LIKE 'A%';

-- Termina con 'ez'
SELECT * FROM usuarios WHERE nombre LIKE '%ez';

-- Contiene 'mar' en cualquier posición
SELECT * FROM usuarios WHERE email LIKE '%mar%';

-- Nombre de exactamente 5 letras
SELECT * FROM usuarios WHERE nombre LIKE '_____';

-- Email que termina en gmail.com
SELECT * FROM usuarios WHERE email LIKE '%@gmail.com';

-- ILIKE (insensible a mayúsculas/minúsculas) - PostgreSQL
SELECT * FROM usuarios WHERE nombre ILIKE '%martin%';
```

### 3.5 IS NULL / IS NOT NULL

```sql
-- Usuarios sin teléfono registrado
SELECT * FROM usuarios WHERE telefono IS NULL;

-- Usuarios con email registrado
SELECT * FROM usuarios WHERE email IS NOT NULL;

-- Combinado con otros filtros
SELECT * FROM usuarios 
WHERE telefono IS NULL 
AND activo = true;
```

### 3.6 Operadores Lógicos (AND, OR, NOT)

```sql
-- AND: todas las condiciones deben cumplirse
SELECT * FROM usuarios 
WHERE edad >= 18 
AND edad <= 30 
AND ciudad = 'Madrid';

-- OR: al menos una condición debe cumplirse
SELECT * FROM usuarios 
WHERE ciudad = 'Madrid' 
OR ciudad = 'Barcelona' 
OR ciudad = 'Valencia';

-- NOT: negación
SELECT * FROM usuarios 
WHERE NOT ciudad = 'Madrid';

-- Combinación compleja
SELECT * FROM pedidos 
WHERE (estado = 'pendiente' OR estado = 'procesando')
AND total > 500
AND fecha >= '2024-01-01'
AND NOT cliente_id IS NULL;
```

---

## Módulo 4: Ordenamiento y Funciones Básicas

### 4.1 ORDER BY

Ordena los resultados según una o más columnas:

```sql
-- Orden ascendente (por defecto)
SELECT * FROM usuarios ORDER BY nombre;

-- Orden descendente
SELECT * FROM usuarios ORDER BY edad DESC;

-- Múltiples columnas
SELECT * FROM usuarios 
ORDER BY ciudad ASC, nombre ASC;

-- Orden con NULLS FIRST/LAST (PostgreSQL)
SELECT * FROM usuarios 
ORDER BY telefono NULLS LAST;
```

### 4.2 Funciones de Texto

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| `UPPER(texto)` | Convertir a mayúsculas | `UPPER('hola') → 'HOLA'` |
| `LOWER(texto)` | Convertir a minúsculas | `LOWER('HOLA') → 'hola'` |
| `LENGTH(texto)` | Longitud del texto | `LENGTH('hola') → 4` |
| `TRIM(texto)` | Eliminar espacios | `TRIM(' hola ') → 'hola'` |
| `SUBSTRING(texto, inicio, largo)` | Extraer parte | `SUBSTRING('hola', 2, 2) → 'ol'` |
| `REPLACE(texto, buscar, reemplazar)` | Reemplazar texto | `REPLACE('hola', 'o', 'a') → 'hala'` |
| `POSITION(subtexto IN texto)` | Posición de subtexto | `POSITION('l' IN 'hola') → 3` |
| `CONCAT(texto1, texto2)` | Concatenar | `CONCAT('hola', ' mundo')` |

```sql
-- Ejemplos prácticos
SELECT 
    nombre,
    UPPER(nombre) AS nombre_mayusculas,
    LOWER(email) AS email_minusculas,
    LENGTH(TRIM(nombre)) AS longitud_nombre,
    SUBSTRING(email, POSITION('@' IN email) + 1) AS dominio,
    REPLACE(telefono, ' ', '') AS telefono_sin_espacios
FROM usuarios;
```

### 4.3 Funciones de Fecha

| Función | Descripción | Ejemplo |
|---------|-------------|---------|
| `NOW()` | Fecha y hora actual | `NOW()` |
| `CURRENT_DATE` | Fecha actual | `CURRENT_DATE` |
| `CURRENT_TIME` | Hora actual | `CURRENT_TIME` |
| `EXTRACT(part FROM fecha)` | Extraer parte de fecha | `EXTRACT(YEAR FROM fecha)` |
| `DATE_PART('part', fecha)` | Extraer parte de fecha | `DATE_PART('month', fecha)` |
| `AGE(fecha1, fecha2)` | Diferencia entre fechas | `AGE('2024-12-31', '2024-01-01')` |
| `DATE_TRUNC('part', fecha)` | Truncar fecha | `DATE_TRUNC('month', fecha)` |
| `fecha + INTERVAL` | Sumar intervalo | `fecha + INTERVAL '1 month'` |

```sql
-- Ejemplos prácticos
SELECT 
    nombre,
    fecha_registro,
    EXTRACT(YEAR FROM fecha_registro) AS año_registro,
    EXTRACT(MONTH FROM fecha_registro) AS mes_registro,
    EXTRACT(DOW FROM fecha_registro) AS dia_semana,  -- 0=domingo
    DATE_PART('quarter', fecha_registro) AS trimestre,
    AGE(NOW(), fecha_registro) AS tiempo_registrado,
    fecha_registro + INTERVAL '1 year' AS fecha_aniversario
FROM usuarios;

-- Filtrar por año usando EXTRACT
SELECT * FROM pedidos 
WHERE EXTRACT(YEAR FROM fecha) = 2024;
```

---

## Módulo 5: Introducción a JOINs

Las bases de datos relacionales se basan en la normalización, que distribuye la información en múltiples tablas relacionadas mediante claves. Para recuperar datos que están dispersos, necesitamos combinar tablas mediante **JOINs**. Un JOIN combina filas de dos o más tablas basándose en una condición relacionada.

### 5.1 Producto Cartesiano (CROSS JOIN)

El producto cartesiano combina cada fila de la primera tabla con cada fila de la segunda tabla. Si la tabla A tiene N filas y la tabla B tiene M filas, el resultado tendrá N × M filas.

```sql
-- Producto cartesiano explícito
SELECT * FROM usuarios CROSS JOIN pedidos;

-- Producto cartesiano implícito (sin condición)
SELECT * FROM usuarios, pedidos;
```

**⚠️ Advertencia:** El producto cartesiano raramente es útil por sí solo y puede generar resultados muy grandes. Siempre debe usarse con una condición de JOIN.

### 5.2 INNER JOIN

El **INNER JOIN** devuelve solo las filas que tienen correspondencia en ambas tablas. Es el tipo de JOIN más común.

```sql
-- Sintaxis explícita (recomendada)
SELECT columnas
FROM tablaA
INNER JOIN tablaB ON tablaA.clave = tablaB.clave;

-- Sintaxis implícita (menos recomendada)
SELECT columnas
FROM tablaA, tablaB
WHERE tablaA.clave = tablaB.clave;
```

#### Ejemplo

```sql
-- Obtener mensajes con el nombre del usuario que los envió
SELECT m.id, u.nombre, m.contenido, m.modalidad, m.enviado_en
FROM mensajes m
INNER JOIN usuarios u ON m.remitente_id = u.id;

-- Obtener sesiones con información del usuario
SELECT s.id, u.nombre, s.canal, s.iniciada_en, s.ultima_actividad
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id;

-- Obtener logs con el mensaje asociado (JOIN de 2 tablas)
SELECT l.modulo, l.tiempo_ejecucion_ms, l.exito, m.contenido
FROM logs_ejecucion l
INNER JOIN mensajes m ON l.mensaje_id = m.id;
```

### 5.3 LEFT JOIN (LEFT OUTER JOIN)

El **LEFT JOIN** devuelve **todas** las filas de la tabla izquierda, y las filas coincidentes de la tabla derecha. Si no hay coincidencia, las columnas de la tabla derecha se rellenan con NULL.

```sql
SELECT columnas
FROM tablaA
LEFT JOIN tablaB ON tablaA.clave = tablaB.clave;
```

#### Ejemplo con el caso Aletza

```sql
-- Todos los usuarios, incluso aquellos sin perfil de voz
SELECT u.nombre, u.email, p.estado, p.muestras_tomadas
FROM usuarios u
LEFT JOIN perfiles_voz p ON u.id = p.usuario_id;

-- Resultado: muestra todos los usuarios, con NULL en perfil si no existe

-- Todos los usuarios, incluso aquellos sin sesiones
SELECT u.nombre, COUNT(s.id) AS numero_sesiones
FROM usuarios u
LEFT JOIN sesiones s ON u.id = s.usuario_id
GROUP BY u.id, u.nombre;

-- Todos los mensajes, incluso aquellos sin logs de ejecución
SELECT m.id, m.contenido, l.modulo, l.tiempo_ejecucion_ms
FROM mensajes m
LEFT JOIN logs_ejecucion l ON m.id = l.mensaje_id;
```

### 5.4 RIGHT JOIN

El **RIGHT JOIN** es simétrico al LEFT JOIN: devuelve todas las filas de la tabla derecha, y las coincidentes de la izquierda.

```sql
SELECT columnas
FROM tablaA
RIGHT JOIN tablaB ON tablaA.clave = tablaB.clave;
```

**Nota:** RIGHT JOIN puede reescribirse como LEFT JOIN intercambiando el orden de las tablas. Por claridad, se recomienda usar LEFT JOIN.

```sql
-- Estos dos son equivalentes:
SELECT * FROM usuarios u RIGHT JOIN sesiones s ON u.id = s.usuario_id;
SELECT * FROM sesiones s LEFT JOIN usuarios u ON u.id = s.usuario_id;
```

### 5.5 JOIN con Múltiples Tablas

Podemos encadenar varios JOINs para combinar información de múltiples tablas:

```sql
SELECT columnas
FROM tablaA
JOIN tablaB ON tablaA.clave = tablaB.clave
JOIN tablaC ON tablaB.clave2 = tablaC.clave2;
```

#### Ejemplo con el caso Aletza (3 tablas)

```sql
-- Obtener mensajes con usuario y sus logs de ejecución
SELECT 
    u.nombre AS usuario,
    m.contenido AS mensaje,
    m.modalidad,
    l.modulo AS modulo_ia,
    l.tiempo_ejecucion_ms,
    l.exito
FROM mensajes m
INNER JOIN usuarios u ON m.remitente_id = u.id
LEFT JOIN logs_ejecucion l ON m.id = l.mensaje_id
ORDER BY m.enviado_en DESC;

-- Obtener sesiones activas con información completa
SELECT 
    s.id AS sesion_id,
    u.nombre AS usuario,
    u.email,
    s.canal,
    s.iniciada_en,
    s.ultima_actividad,
    COUNT(m.id) AS mensajes_en_sesion
FROM sesiones s
INNER JOIN usuarios u ON s.usuario_id = u.id
LEFT JOIN mensajes m ON s.id = m.sesion_id
WHERE s.expira_en > NOW()  -- sesiones no expiradas
GROUP BY s.id, u.nombre, u.email, s.canal, s.iniciada_en, s.ultima_actividad;
```

### 5.6 Self-Join (Auto-Join)

Un **Self-Join** ocurre cuando una tabla se une consigo misma. Es útil para relaciones jerárquicas o cuando necesitamos comparar filas dentro de la misma tabla.

```sql
-- Es obligatorio usar alias diferentes para la misma tabla
SELECT 
    a.columna1,
    b.columna2
FROM tabla a
JOIN tabla b ON a.clave = b.clave_referencia;
```

#### Ejemplo

Supongamos que tenemos una tabla de módulos IA que pueden llamar a otros módulos:

```sql
-- Tabla de módulos IA (ejemplo adicional)
CREATE TABLE modulos_ia (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(50),
    version VARCHAR(10),
    modulo_padre_id INTEGER REFERENCES modulos_ia(id)
);

-- Self-Join para obtener jerarquía de módulos
SELECT 
    hijo.nombre AS modulo_hijo,
    padre.nombre AS modulo_padre,
    hijo.version
FROM modulos_ia hijo
LEFT JOIN modulos_ia padre ON hijo.modulo_padre_id = padre.id
ORDER BY padre.nombre NULLS FIRST;

-- Encontrar módulos que se ejecutan más rápido que el promedio
SELECT 
    m1.modulo,
    m1.tiempo_ejecucion_ms,
    m2.tiempo_promedio
FROM logs_ejecucion m1
JOIN (
    SELECT modulo, AVG(tiempo_ejecucion_ms) AS tiempo_promedio
    FROM logs_ejecucion
    GROUP BY modulo
) m2 ON m1.modulo = m2.modulo
WHERE m1.tiempo_ejecucion_ms < m2.tiempo_promedio;
```

### 5.7 JOIN con Condiciones No Equitativas

Las condiciones de JOIN no tienen que ser necesariamente de igualdad (`=`). Podemos usar otros operadores como `<`, `>`, `BETWEEN`, etc.

```sql
-- JOIN con condición de desigualdad
SELECT * 
FROM tablaA a
JOIN tablaB b ON a.id <> b.id;

-- JOIN con condición de rango
SELECT * 
FROM ventas v
JOIN productos p ON v.precio BETWEEN p.precio_minimo AND p.precio_maximo;

-- JOIN con condición de fecha
SELECT * 
FROM empleados e
JOIN contrataciones c ON e.fecha_ingreso < c.fecha_corte;
```

#### Ejemplo con el caso Aletza

```sql
-- Encontrar mensajes que fueron procesados más rápido que el promedio de su módulo
SELECT 
    m.id,
    m.contenido,
    l.modulo,
    l.tiempo_ejecucion_ms,
    p.tiempo_promedio_modulo
FROM logs_ejecucion l
JOIN mensajes m ON l.mensaje_id = m.id
JOIN (
    SELECT modulo, AVG(tiempo_ejecucion_ms) AS tiempo_promedio_modulo
    FROM logs_ejecucion
    GROUP BY modulo
) p ON l.modulo = p.modulo
WHERE l.tiempo_ejecucion_ms < p.tiempo_promedio_modulo
ORDER BY (p.tiempo_promedio_modulo - l.tiempo_ejecucion_ms) DESC;

-- Encontrar sesiones que tuvieron más mensajes que el promedio
SELECT 
    s.id,
    s.canal,
    COUNT(m.id) AS mensajes_sesion
FROM sesiones s
LEFT JOIN mensajes m ON s.id = m.sesion_id
GROUP BY s.id, s.canal
HAVING COUNT(m.id) > (
    SELECT AVG(mensajes_por_sesion)
    FROM (
        SELECT COUNT(m2.id) AS mensajes_por_sesion
        FROM sesiones s2
        LEFT JOIN mensajes m2 ON s2.id = m2.sesion_id
        GROUP BY s2.id
    ) AS promedios
);
```

