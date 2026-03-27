
## Sesión 4
# MANIPULACIÓN DE DATOS Y LÓGICA DE CONJUNTOS
## DML, JOINs, Subconsultas, CTEs y Optimización en SQL

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción a la Manipulación de Datos

En las sesiones anteriores hemos aprendido a modelar bases de datos, normalizar y definir estructuras. Ahora es el momento de interactuar con los datos: insertarlos, consultarlos, modificarlos y eliminarlos. El lenguaje SQL (Structured Query Language) es el estándar para estas tareas. En esta sesión nos centraremos en el sublenguaje DML (Data Manipulation Language) y en las operaciones de conjuntos que nos permiten combinar información de múltiples tablas: los JOINs. También exploraremos cómo el motor de base de datos interpreta y ejecuta nuestras consultas, qué sucede a nivel hardware, y cómo escribir SQL eficiente. Dominar estos conceptos es fundamental para cualquier profesional que trabaje con datos, ya que constituyen la base de cualquier análisis, informe o aplicación, y son temas recurrentes en entrevistas técnicas nacionales e internacionales.

## El Lenguaje SQL y sus Sublenguajes

SQL se divide en varios sublenguajes según su función:

- **DDL (Data Definition Language):** CREATE, ALTER, DROP -- definen la estructura.

- **DML (Data Manipulation Language):** SELECT, INSERT, UPDATE, DELETE -- manipulan los datos.

- **DCL (Data Control Language):** GRANT, REVOKE -- controlan permisos.

- **TCL (Transaction Control Language):** COMMIT, ROLLBACK -- gestionan transacciones.

En esta sesión profundizaremos en DML, con especial énfasis en SELECT y JOINs.

# Lenguaje SQL: Sublenguajes y Operaciones Fundamentales

El lenguaje SQL se divide en varios sublenguajes según la naturaleza de las operaciones. Conocer cada uno y sus comandos es esencial para cualquier profesional de bases de datos, ya que permite definir estructuras, manipular datos, controlar permisos y gestionar transacciones. En este capítulo exploraremos los cuatro sublenguajes principales: DDL, DML, DCL y TCL, con ejemplos prácticos y casos de uso que suelen aparecer en entrevistas técnicas.

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

También se pueden crear índices, vistas, etc.:

```sql
CREATE INDEX idx_ciudad ON clientes(ciudad);
CREATE VIEW clientes_madrid AS SELECT * FROM clientes WHERE ciudad = 'Madrid';
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

Ejemplo:

```sql
SELECT nombre, email FROM clientes WHERE ciudad = 'Barcelona' ORDER BY nombre;
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

Algunos motores (SQL Server, Oracle, PostgreSQL con ON CONFLICT) permiten insertar o actualizar según exista o no la fila. Es muy útil en sincronización de datos.

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

### Casos de Uso en Entrevistas

En entrevistas suelen preguntar cómo se gestionan los permisos en entornos multi-rol, o cómo asegurar que un usuario solo pueda leer datos. También es común preguntar sobre el principio de mínimo privilegio.

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

### Propiedades ACID

Las transacciones garantizan las propiedades ACID (Atomicidad, Consistencia, Aislamiento, Durabilidad), tema recurrente en entrevistas. Es importante saber explicarlas con ejemplos.

## Comparativa entre Sublenguajes

A modo de resumen, la siguiente tabla muestra los comandos principales de cada sublenguaje:

::: center
  **Sublenguaje**   **Comandos principales**                **Uso**
  ----------------- --------------------------------------- -------------------------
  DDL               CREATE, ALTER, DROP, TRUNCATE           Definir estructura
  DML               SELECT, INSERT, UPDATE, DELETE, MERGE   Manipular datos
  DCL               GRANT, REVOKE                           Controlar permisos
  TCL               BEGIN, COMMIT, ROLLBACK, SAVEPOINT      Gestionar transacciones
:::

## Ejercicios Resueltos Integradores

A continuación, ejercicios que combinan varios sublenguajes.

### Ejercicio 1: Creación y manipulación

**Enunciado:** Crear una tabla `productos` con columnas id, nombre, precio, stock. Insertar tres productos. Actualizar el precio de uno. Eliminar un producto. Finalmente, otorgar permisos de SELECT a un usuario.

**Solución:**

```sql
-- DDL
CREATE TABLE productos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio DECIMAL(10,2) CHECK (precio > 0),
    stock INT DEFAULT 0
);

-- DML
INSERT INTO productos (nombre, precio, stock) VALUES
('Laptop', 1200.00, 10),
('Mouse', 25.50, 50),
('Teclado', 45.00, 30);

UPDATE productos SET precio = 1150.00 WHERE nombre = 'Laptop';

DELETE FROM productos WHERE nombre = 'Mouse';

-- DCL
GRANT SELECT ON productos TO analista;
```

### Ejercicio 2: Transacción con error

**Enunciado:** Realizar una transferencia bancaria entre dos cuentas, asegurando que si una operación falla, ninguna se aplique. Usar transacciones.

**Solución:**

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 500 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 500 WHERE id = 2;
-- Si todo va bien
COMMIT;
-- Si ocurre un error antes del COMMIT, se puede hacer ROLLBACK
```

### Ejercicio 3: Permisos y seguridad

**Enunciado:** Crear un usuario 'lector' que solo pueda hacer SELECT en la tabla `clientes`, y un usuario 'editor' que pueda INSERT, UPDATE y DELETE. Implementar los comandos DCL.

**Solución:**

```sql
-- Crear usuarios (en PostgreSQL)
CREATE USER lector WITH PASSWORD 'pass1';
CREATE USER editor WITH PASSWORD 'pass2';

-- Otorgar permisos
GRANT SELECT ON clientes TO lector;
GRANT SELECT, INSERT, UPDATE, DELETE ON clientes TO editor;

-- Revocar permisos si es necesario
REVOKE DELETE ON clientes FROM editor;
```

## Conclusión

Dominar los cuatro sublenguajes de SQL es fundamental para cualquier profesional de datos. No solo se trata de escribir consultas, sino de entender cómo definir estructuras robustas (DDL), manipular datos eficientemente (DML), controlar accesos (DCL) y garantizar la integridad transaccional (TCL). Estos conceptos son evaluados constantemente en entrevistas técnicas y son la base para construir sistemas de bases de datos seguros, escalables y confiables.

# El Proceso Interno de una Consulta SQL

Para escribir SQL eficiente, es crucial entender cómo el motor procesa una consulta. No basta con saber la sintaxis; hay que comprender el flujo lógico y físico.

## Orden Lógico de Ejecución de una Consulta

Aunque escribimos SELECT \... FROM \... WHERE \... GROUP BY \... HAVING \... ORDER BY \... LIMIT, el motor ejecuta en un orden diferente. El orden lógico (conceptual) es:

1.  **FROM y JOINs**: Se construye el conjunto de datos base (producto cartesiano de las tablas involucradas, aplicando las condiciones de JOIN).

2.  **WHERE**: Se filtran las filas según las condiciones.

3.  **GROUP BY**: Se agrupan las filas.

4.  **HAVING**: Se filtran los grupos.

5.  **SELECT**: Se evalúan las expresiones de las columnas, incluyendo funciones de agregación.

6.  **ORDER BY**: Se ordena el resultado.

7.  **LIMIT / OFFSET**: Se recorta el resultado.

Este orden es importante porque, por ejemplo, no se puede usar un alias de SELECT en WHERE (porque WHERE se ejecuta antes). Tampoco se pueden usar funciones de ventana en WHERE por la misma razón.

### Ejemplo

```sql
SELECT nombre, salario, salario * 12 AS salario_anual
FROM empleados
WHERE salario > 3000
ORDER BY salario_anual;
```

Aquí, el alias `salario_anual` se puede usar en ORDER BY porque se calcula antes de ordenar, pero no se podría usar en WHERE.

## El Camino Físico: De la Consulta al Hardware

Cuando ejecutamos una consulta, ocurren múltiples etapas:

### Parsing (Análisis Sintáctico)

El motor verifica que la consulta sea sintácticamente correcta y que las tablas y columnas existan. Genera un árbol de consulta.

### Optimización

El optimizador (basado en costos, CBO) genera varios planes de ejecución posibles y estima su costo (en términos de E/S, CPU, memoria). Considera:

- Estadísticas de las tablas (número de filas, distribución de valores, histogramas).

- Existencia de índices.

- Métodos de acceso (seq scan, index scan, bitmap scan).

- Algoritmos de JOIN (nested loop, hash join, merge join).

Elige el plan de menor costo estimado.

### Ejecución

El plan se ejecuta, interactuando con el buffer cache y el almacenamiento. Si los datos están en caché (buffer pool), se evita E/S a disco. De lo contrario, se leen páginas del disco.

### Retorno de Resultados

Las filas resultantes se envían al cliente, posiblemente en lotes.

### Hardware Involucrado

- **CPU**: Procesa las condiciones, cálculos, agregaciones.

- **RAM (Buffer Pool)**: Almacena páginas de datos e índices recientemente usados.

- **Disco (SSD/HDD)**: Si hay cache miss, se lee de disco.

- **Red**: Transmite los resultados al cliente.

En entrevistas, preguntan cómo optimizar: reducir E/S, usar índices, evitar ordenamientos innecesarios, etc.

## EXPLAIN: La Herramienta Clave

Para entender el plan de ejecución, usamos `EXPLAIN` (y `EXPLAIN ANALYZE` para ejecutar realmente y ver tiempos).

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM clientes WHERE ciudad = 'Madrid';
```

Muestra el plan, costos estimados, filas reales, tiempo, y accesos a buffers (caché vs disco). Es esencial en entrevistas para demostrar capacidad de diagnóstico.

# Joins: Combinando Tablas

Las bases de datos relacionales se basan en la normalización, que distribuye la información en múltiples tablas relacionadas mediante claves. Para recuperar datos que están dispersos, necesitamos combinar tablas mediante JOINs. Un JOIN combina filas de dos o más tablas basándose en una condición relacionada.

## Producto Cartesiano y JOIN

Si escribimos:

```sql
SELECT * FROM A, B;
```

Obtenemos el producto cartesiano (todas las combinaciones de filas de A con B). Esto rara vez es útil. Los JOINs añaden una condición para filtrar las combinaciones relevantes.

## Tipos de JOIN

Existen varios tipos, que se ilustran con diagramas de Venn (aunque hay que tener cuidado porque los conjuntos no son exactamente así, son ilustrativos).

### INNER JOIN

Devuelve solo las filas que tienen correspondencia en ambas tablas.

::: center
:::

Sintaxis:

```sql
SELECT columnas
FROM tablaA
INNER JOIN tablaB ON tablaA.clave = tablaB.clave;
```

También se puede usar la palabra clave `JOIN` a secas (por defecto es INNER).

### LEFT JOIN (o LEFT OUTER JOIN)

Devuelve todas las filas de la tabla izquierda, y las coincidentes de la derecha. Si no hay coincidencia, las columnas de la derecha son NULL.

::: center
:::

Sintaxis:

```sql
SELECT columnas
FROM tablaA
LEFT JOIN tablaB ON tablaA.clave = tablaB.clave;
```

### RIGHT JOIN

Análogo al LEFT, pero con la tabla derecha como maestra. Es menos usado porque se puede escribir el LEFT cambiando el orden.

### FULL OUTER JOIN

Devuelve todas las filas de ambas tablas, rellenando con NULL donde no hay coincidencia.

::: center
:::

Sintaxis:

```sql
SELECT columnas
FROM tablaA
FULL OUTER JOIN tablaB ON tablaA.clave = tablaB.clave;
```

### CROSS JOIN

Producto cartesiano (cada fila de A con cada fila de B). No lleva condición.

```sql
SELECT * FROM tablaA CROSS JOIN tablaB;
-- Equivalente a SELECT * FROM tablaA, tablaB;
```

### Self-Join

Una tabla se une consigo misma. Es útil para relaciones jerárquicas (ej. empleados con jefe).

```sql
SELECT e1.nombre AS empleado, e2.nombre AS jefe
FROM empleados e1
LEFT JOIN empleados e2 ON e1.jefe_id = e2.id;
```

### JOIN con condiciones no equitativas

La condición puede incluir otros operadores (\>, \<, BETWEEN). Pero hay que tener en cuenta que estos joins pueden ser costosos.

```sql
SELECT *
FROM ventas v
JOIN productos p ON v.producto_id = p.id AND v.cantidad > p.stock_minimo;
```

## Algoritmos de JOIN Internos

El motor elige entre varios algoritmos según el tamaño de las tablas y los índices:

- **Nested Loop:** Para cada fila de la tabla externa, busca coincidencias en la interna (ideal si una es pequeña o hay índice).

- **Hash Join:** Construye una tabla hash en memoria con una de las tablas, luego recorre la otra. Eficiente para tablas grandes sin índices.

- **Merge Join:** Ordena ambas tablas (si no lo están) y luego las fusiona. Útil si ya están ordenadas o se necesita ordenar de todas formas.

Saber cuándo se usa cada uno es una pregunta clásica en entrevistas.

## Ejemplo Práctico con Múltiples Tablas

Supongamos una base de datos de ventas con las tablas:

- `clientes` (id, nombre, ciudad)

- `productos` (id, nombre, precio)

- `pedidos` (id, cliente_id, fecha)

- `detalle_pedido` (pedido_id, producto_id, cantidad)

Queremos obtener una lista de pedidos con nombre del cliente, fecha, productos y cantidad:

```sql
SELECT c.nombre AS cliente, p.fecha, pr.nombre AS producto, dp.cantidad
FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
JOIN detalle_pedido dp ON p.id = dp.pedido_id
JOIN productos pr ON dp.producto_id = pr.id;
```

Este es un JOIN de cuatro tablas. El orden de los JOINs puede afectar el rendimiento, pero el optimizador suele reordenarlos.

# Subconsultas

Una subconsulta es una consulta anidada dentro de otra. Pueden usarse en SELECT, FROM, WHERE, etc. Son herramientas poderosas pero a veces pueden reescribirse como JOINs para mejorar rendimiento.

## Subconsultas Escalares

Devuelven un solo valor (una fila y una columna). Se usan en expresiones.

```sql
SELECT nombre, (SELECT AVG(precio) FROM productos) AS precio_medio
FROM productos;
```

Deben asegurar que devuelven una sola fila; si devuelven varias, se produce error.

## Subconsultas de Fila Única

Devuelven una fila con varias columnas. Se pueden comparar con operadores como `=, >, <` si se espera una sola fila.

```sql
SELECT * FROM productos
WHERE (categoria, precio) = (SELECT categoria, MAX(precio) FROM productos GROUP BY categoria LIMIT 1);
```

(En la práctica, es más seguro usar EXISTS o IN.)

## Subconsultas de Múltiples Filas

Devuelven varias filas. Se usan con `IN`, `ANY`, `ALL`, `EXISTS`.

- **IN**: Verifica si el valor está en el conjunto.

- **ANY / SOME**: Compara con algún valor del conjunto (ej. \> ANY significa mayor que al menos uno).

- **ALL**: Compara con todos los valores (ej. \> ALL significa mayor que todos).

- **EXISTS**: Verifica si la subconsulta devuelve alguna fila (es más eficiente que IN para conjuntos grandes porque puede parar al encontrar la primera).

```sql
SELECT nombre FROM clientes
WHERE id IN (SELECT cliente_id FROM pedidos WHERE fecha >= '2025-01-01');

SELECT nombre FROM clientes c
WHERE EXISTS (SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id AND p.fecha >= '2025-01-01');
```

`EXISTS` suele ser más rápido cuando la subconsulta es correlacionada.

## Subconsultas Correlacionadas

Hacen referencia a columnas de la consulta exterior. Se evalúan para cada fila de la consulta externa.

```sql
SELECT p.nombre, p.precio
FROM productos p
WHERE p.precio > (SELECT AVG(precio) FROM productos WHERE categoria = p.categoria);
```

Aquí, la subconsulta calcula el precio medio de la categoría del producto actual. Pueden ser lentas si la tabla externa es grande.

## Subconsultas en la cláusula FROM

Se puede usar una subconsulta como una tabla derivada (vista temporal). Requiere un alias.

```sql
SELECT categoria, AVG(precio) AS precio_medio
FROM (SELECT * FROM productos WHERE activo = true) AS productos_activos
GROUP BY categoria;
```

# Common Table Expressions (CTEs)

Las CTEs (WITH queries) permiten definir consultas auxiliares que se comportan como tablas temporales dentro de una consulta principal. Mejoran la legibilidad y permiten consultas recursivas.

## Sintaxis Básica

```sql
WITH cte_nombre (columnas_opcional) AS (
    consulta
)
SELECT ...
FROM cte_nombre ...
```

### Ejemplo

Obtener el total de ventas por cliente y luego filtrar los que superan 1000:

```sql
WITH ventas_cliente AS (
    SELECT c.id, c.nombre, SUM(dp.cantidad * pr.precio) AS total
    FROM clientes c
    LEFT JOIN pedidos p ON c.id = p.cliente_id
    LEFT JOIN detalle_pedido dp ON p.id = dp.pedido_id
    LEFT JOIN productos pr ON dp.producto_id = pr.id
    GROUP BY c.id, c.nombre
)
SELECT * FROM ventas_cliente WHERE total > 1000;
```

### CTEs Múltiples

Se pueden definir varias CTEs separadas por comas.

```sql
WITH
    ventas AS (SELECT ...),
    clientes_vip AS (SELECT ...)
SELECT ...
```

### CTEs Recursivas

Permiten consultar estructuras jerárquicas (ej. organigrama). La sintaxis incluye una parte no recursiva (anchor) y una parte recursiva unida con UNION ALL.

```sql
WITH RECURSIVE organigrama AS (
    -- Anchor: el jefe máximo
    SELECT id, nombre, jefe_id, 1 AS nivel
    FROM empleados
    WHERE jefe_id IS NULL
    UNION ALL
    -- Recursivo: empleados que tienen jefe
    SELECT e.id, e.nombre, e.jefe_id, o.nivel + 1
    FROM empleados e
    JOIN organigrama o ON e.jefe_id = o.id
)
SELECT * FROM organigrama;
```

Es una pregunta común en entrevistas avanzadas.

## CTEs vs. Subconsultas vs. Tablas Temporales

- Las CTEs son solo azúcar sintáctico; el optimizador puede tratarlas como subconsultas en línea o materializarlas según el motor (en PostgreSQL, se materializan por defecto a menos que se use `NOT MATERIALIZED`).

- Son más legibles y permiten recursión.

- Para reutilización múltiple en la misma consulta, una CTE evita repetir código.

# Buenas Prácticas y Optimización de Consultas SQL

Escribir SQL eficiente no solo es cuestión de sintaxis correcta, sino de entender cómo el motor ejecutará la consulta. Aquí hay reglas y patrones que suelen preguntar en entrevistas.

## Preferir EXISTS a IN cuando la subconsulta es grande

```sql
-- Menos eficiente si la subconsulta devuelve muchas filas
SELECT * FROM clientes WHERE id IN (SELECT cliente_id FROM pedidos);

-- Más eficiente (puede parar al encontrar el primero)
SELECT * FROM clientes c WHERE EXISTS (SELECT 1 FROM pedidos p WHERE p.cliente_id = c.id);
```

## Evitar Funciones en Columnas Indexadas

```sql
-- No usará índice en fecha
SELECT * FROM pedidos WHERE EXTRACT(YEAR FROM fecha) = 2025;

-- Mejor (si hay índice en fecha)
SELECT * FROM pedidos WHERE fecha >= '2025-01-01' AND fecha < '2026-01-01';
```

## Usar UNION ALL en lugar de UNION si no se necesitan duplicados

UNION elimina duplicados (requiere ordenar y comparar), mientras que UNION ALL simplemente concatena.

## SELECT \* Es Peligroso

Siempre especificar las columnas necesarias. SELECT \* puede traer más datos de los necesarios, aumentar E/S y red, y romper la aplicación si cambia el esquema.

## Cuidado con las Conversiones Implícitas

Si comparas un texto con un número, el motor puede convertir toda la columna, impidiendo el uso de índices.

```sql
-- Si id es texto, pero lo comparas con número, puede haber conversión
SELECT * FROM tabla WHERE id = 123; -- mejor usar '123'
```

## Índices y JOINs

Asegurar que las columnas de JOIN estén indexadas. Para JOINs de muchos a muchos, indexar ambas columnas en la tabla intermedia.

## Analizar el Plan con EXPLAIN

Siempre que una consulta sea lenta, usar EXPLAIN (ANALYZE) para ver dónde está el cuello de botella.

## Escritura de Consultas Legibles

Usar indentación, alias claros, y mayúsculas para palabras clave. No afecta el rendimiento pero facilita el mantenimiento.

# Consideraciones de Hardware y Motor

## Buffer Pool y Caché

El tamaño del buffer pool (por ejemplo, `innodb_buffer_pool_size` en MySQL, `shared_buffers` en PostgreSQL) es crítico. Si los datos caben en memoria, las consultas son mucho más rápidas.

## Disco: SSD vs HDD

En producción, siempre SSD. Las lecturas secuenciales son más rápidas, pero los índices requieren lecturas aleatorias. SSD mejora drásticamente las lecturas aleatorias.

## Parallel Query

Algunos motores pueden usar múltiples CPUs para una consulta (paralelismo). Esto puede acelerar agregaciones y escaneos.

## Configuración del Motor

Parámetros como el tamaño de memoria para hash joins, el costo de E/S, etc., pueden ajustarse. En entrevistas, suelen preguntar sobre parámetros genéricos.

# Breve Comparación entre Motores (para JOINs y DML)

Aunque nos centramos en SQL estándar, cada motor tiene peculiaridades:

- **PostgreSQL**: Cumple con el estándar, soporta CTEs recursivas, FULL JOIN, y tiene un optimizador muy maduro.

- **MySQL**: No soporta FULL JOIN (se simula con UNION). Los JOINs pueden ser lentos si no se usan índices.

- **SQL Server**: Ofrece sugerencias de JOIN (LOOP, HASH, MERGE) y soporta CTEs recursivas.

- **Oracle**: Sintaxis antigua de JOIN (+) para outer joins, pero se recomienda usar ANSI.

En entrevistas, es importante conocer las diferencias si el puesto requiere un motor específico.

# Joins en el Mundo NoSQL (Una Breve Mirada)

Aunque esta sesión se centra en SQL, es útil saber cómo se abordan las relaciones en bases de datos NoSQL, ya que en proyectos modernos pueden convivir ambos paradigmas.

- **MongoDB (documental):** No soporta joins nativos entre colecciones, pero desde la versión 3.2 incluye el operador `$lookup` en el pipeline de agregación, que realiza un left outer join con otra colección. Sin embargo, su uso es limitado y se recomienda diseñar documentos embebidos para evitar joins.

- **Cassandra (columna ancha):** No soporta joins. El modelado se basa en desnormalización y tablas diseñadas específicamente para las consultas.

- **Redis (clave-valor):** No tiene joins; las relaciones se manejan a nivel de aplicación o mediante estructuras como sets y sorted sets.

- **Neo4j (grafos):** Las relaciones son nativas; en lugar de joins, se navegan los nodos y relaciones con patrones de camino.

En resumen, en NoSQL se sacrifica la capacidad de join en favor de escalabilidad y rendimiento, por lo que el diseño de datos es fundamental.

# Ejercicios Resueltos

A continuación se presentan ejercicios que integran los conceptos vistos. Se resuelven paso a paso.

## Ejercicio 1: Consultas Básicas con DML

**Enunciado:** Dada la tabla `empleados` con columnas `id`, *nombre*, *salario*, *departamento_id*. Insertar tres empleados, actualizar el salario de uno, y luego eliminar los empleados de un departamento específico.

**Solución:**

```sql
-- Insertar
INSERT INTO empleados (nombre, salario, departamento_id) VALUES
('Juan Perez', 3000, 1),
('Ana Garcia', 3500, 2),
('Luis Fernandez', 3200, 1);

-- Actualizar salario de Juan
UPDATE empleados SET salario = 3300 WHERE nombre = 'Juan Perez';

-- Eliminar empleados del departamento 2
DELETE FROM empleados WHERE departamento_id = 2;
```

## Ejercicio 2: INNER JOIN y LEFT JOIN

**Enunciado:** Usar las tablas `clientes`, `pedidos` y `detalle_pedido` del ejemplo anterior. Obtener:

1.  Todos los pedidos con el nombre del cliente y la fecha.

2.  Todos los clientes, incluso si no tienen pedidos, mostrando el número de pedidos (0 si no tiene).

3.  El total gastado por cada cliente (suma de cantidad \* precio de cada producto en sus pedidos).

**Solución:**

```sql
-- 1. Pedidos con cliente
SELECT p.id, c.nombre, p.fecha
FROM pedidos p
INNER JOIN clientes c ON p.cliente_id = c.id;

-- 2. Todos los clientes con número de pedidos
SELECT c.id, c.nombre, COUNT(p.id) AS num_pedidos
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre;

-- 3. Total gastado por cliente
SELECT c.id, c.nombre, COALESCE(SUM(dp.cantidad * pr.precio), 0) AS total_gastado
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id
LEFT JOIN detalle_pedido dp ON p.id = dp.pedido_id
LEFT JOIN productos pr ON dp.producto_id = pr.id
GROUP BY c.id, c.nombre;
```

Explicación: Usamos LEFT JOIN para incluir clientes sin pedidos, y COALESCE para mostrar 0 en lugar de NULL.

## Ejercicio 3: Subconsultas

**Enunciado:** Encontrar los productos cuyo precio es superior al precio medio de su categoría. Usar subconsulta correlacionada.

**Solución:**

```sql
SELECT p.nombre, p.precio, p.categoria
FROM productos p
WHERE p.precio > (SELECT AVG(precio) FROM productos WHERE categoria = p.categoria);
```

## Ejercicio 4: Uso de CTE

**Enunciado:** Obtener los empleados que ganan más que el salario promedio de su departamento, mostrando el nombre del empleado, su salario y el promedio del departamento.

**Solución:**

```sql
WITH promedio_depto AS (
    SELECT departamento_id, AVG(salario) AS salario_prom
    FROM empleados
    GROUP BY departamento_id
)
SELECT e.nombre, e.salario, pd.salario_prom
FROM empleados e
JOIN promedio_depto pd ON e.departamento_id = pd.departamento_id
WHERE e.salario > pd.salario_prom;
```

## Ejercicio 5: Optimización y EXPLAIN

**Enunciado:** Dada una consulta lenta, ¿cómo diagnosticarías y optimizarías? Supongamos: `SELECT * FROM pedidos WHERE EXTRACT(YEAR FROM fecha) = 2025;`

**Solución:**

1.  Usar EXPLAIN ANALYZE para ver el plan.

2.  Probablemente hará un seq scan porque la función impide usar índice en fecha.

3.  Reescribir: `SELECT * FROM pedidos WHERE fecha >= ’2025-01-01’ AND fecha < ’2026-01-01’;`

4.  Crear índice en `fecha` si no existe.

5.  Verificar con EXPLAIN que ahora use index scan.

## Ejercicio 6: Recursividad con CTE

**Enunciado:** Dada una tabla `empleados` con `id`, *nombre*, *jefe_id*, obtener el organigrama completo con niveles.

**Solución:**

```sql
WITH RECURSIVE organigrama AS (
    SELECT id, nombre, jefe_id, 1 AS nivel
    FROM empleados
    WHERE jefe_id IS NULL
    UNION ALL
    SELECT e.id, e.nombre, e.jefe_id, o.nivel + 1
    FROM empleados e
    JOIN organigrama o ON e.jefe_id = o.id
)
SELECT * FROM organigrama ORDER BY nivel, nombre;
```

# Glosario de Términos

DML

:   Data Manipulation Language: sublenguaje de SQL para manipular datos (SELECT, INSERT, UPDATE, DELETE).

JOIN

:   Operación que combina filas de dos o más tablas basándose en una condición relacionada.

INNER JOIN

:   Devuelve solo las filas que tienen correspondencia en ambas tablas.

LEFT JOIN

:   Devuelve todas las filas de la tabla izquierda, y las coincidentes de la derecha.

FULL OUTER JOIN

:   Devuelve todas las filas de ambas tablas, rellenando con NULL donde no hay coincidencia.

Subconsulta

:   Consulta anidada dentro de otra consulta.

Subconsulta correlacionada

:   Subconsulta que hace referencia a columnas de la consulta exterior.

CTE

:   Common Table Expression (WITH clause): consulta auxiliar que puede ser referenciada múltiples veces.

Recursive CTE

:   CTE que se llama a sí misma, útil para jerarquías.

EXPLAIN

:   Comando para mostrar el plan de ejecución de una consulta.

Plan de ejecución

:   Secuencia de operaciones que el motor realizará para ejecutar una consulta.

Buffer Pool

:   Área de memoria donde se cachean páginas de datos e índices.

Index Scan

:   Acceso a tabla mediante un índice.

Seq Scan

:   Escaneo secuencial de toda la tabla.
