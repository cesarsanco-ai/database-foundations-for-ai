
## Sesión 5
# ANÁLISIS AVANZADO Y TRANSFORMACIÓN DE DATOS
## Agregaciones, Ventanas, Limpieza y Técnicas ETL con SQL

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción

En la sesión anterior dominamos la manipulación básica de datos y las operaciones de conjuntos mediante JOINs. Ahora daremos un paso adelante para explorar técnicas analíticas y de transformación que nos permitirán resumir, ordenar, limpiar y preparar datos para análisis más profundos, incluyendo su uso en procesos ETL y alimentación de modelos de inteligencia artificial. Esta sesión cubre agregaciones, funciones de ventana, limpieza de datos (texto, fechas, nulos), tablas temporales, CTEs y técnicas avanzadas como pivotes y expresiones regulares. Cada concepto se acompaña de ejemplos prácticos y ejercicios resueltos que refuerzan el aprendizaje y preparan para entrevistas técnicas donde estos temas son recurrentes.

# Agregaciones y Agrupamientos

Las funciones de agregación permiten resumir conjuntos de filas en un único valor. Combinadas con la cláusula GROUP BY, podemos obtener resúmenes por grupos.

## Funciones de Agregación Básicas

Las funciones de agregación estándar en SQL son:

- `COUNT(*)`: número de filas.

- `COUNT(expr)`: número de valores no nulos de expr.

- `SUM(expr)`: suma de valores.

- `AVG(expr)`: promedio.

- `MIN(expr)`: mínimo.

- `MAX(expr)`: máximo.

Ejemplo: Obtener el número total de clientes y el salario promedio por departamento.

```sql
SELECT departamento_id, COUNT(*) AS num_empleados, AVG(salario) AS salario_promedio
FROM empleados
GROUP BY departamento_id;
```

## GROUP BY con Múltiples Columnas

Podemos agrupar por varias columnas para obtener resúmenes más detallados.

```sql
SELECT departamento_id, ciudad, COUNT(*) AS num_empleados
FROM empleados
GROUP BY departamento_id, ciudad;
```

## Filtrado de Grupos con HAVING

HAVING es similar a WHERE pero aplica a grupos, no a filas individuales. Se usa después de GROUP BY.

```sql
SELECT departamento_id, AVG(salario) AS salario_promedio
FROM empleados
GROUP BY departamento_id
HAVING AVG(salario) > 4000;
```

## Agregaciones Avanzadas: ROLLUP, CUBE y GROUPING SETS

Estas extensiones permiten generar subtotales y totales generales en una misma consulta.

### ROLLUP

Genera subtotales para cada nivel de agrupación, además del total general.

```sql
SELECT departamento_id, ciudad, SUM(salario) AS total_salarios
FROM empleados
GROUP BY ROLLUP (departamento_id, ciudad);
```

Produce filas para cada (depto, ciudad), cada depto (subtotal) y un total general.

### CUBE

Genera todas las combinaciones posibles de subtotales.

```sql
SELECT departamento_id, ciudad, SUM(salario)
FROM empleados
GROUP BY CUBE (departamento_id, ciudad);
```

Incluye subtotales por depto, por ciudad, y total general.

### GROUPING SETS

Permite especificar exactamente los niveles de agrupación deseados.

```sql
SELECT departamento_id, ciudad, SUM(salario)
FROM empleados
GROUP BY GROUPING SETS ((departamento_id), (ciudad), ());
```

El conjunto vacío () representa el total general.

## Ejercicio Resuelto: Resumen de Ventas

**Enunciado:** Dada una tabla `ventas` con columnas `producto`, *categoria*, *monto*, *fecha*, obtener:

1.  Monto total por categoría.

2.  Monto total por categoría y producto.

3.  Solo las categorías cuyo monto total supere 10000.

4.  Además, incluir un total general y subtotales por categoría usando ROLLUP.

**Solución:**

```sql
-- 1. Monto total por categoría
SELECT categoria, SUM(monto) AS total
FROM ventas
GROUP BY categoria;

-- 2. Por categoría y producto
SELECT categoria, producto, SUM(monto) AS total
FROM ventas
GROUP BY categoria, producto;

-- 3. Categorías con total > 10000
SELECT categoria, SUM(monto) AS total
FROM ventas
GROUP BY categoria
HAVING SUM(monto) > 10000;

-- 4. Con ROLLUP
SELECT categoria, producto, SUM(monto) AS total
FROM ventas
GROUP BY ROLLUP (categoria, producto);
```

# Funciones de Ventana (Window Functions)

Las funciones de ventana realizan cálculos sobre un conjunto de filas relacionadas con la fila actual, sin colapsar el resultado en una sola fila. Son esenciales para análisis de series temporales, rankings, y más.

## Concepto y Sintaxis

Una función de ventana se define con la cláusula `OVER()`, que puede incluir:

- `PARTITION BY`: divide el conjunto en particiones.

- `ORDER BY`: ordena las filas dentro de cada partición.

- `ROWS / RANGE`: define el marco de ventana (filas a incluir).

Sintaxis general:

```sql
funcion() OVER (PARTITION BY col1 ORDER BY col2 ROWS BETWEEN ...)
```

## Funciones de Ranking

- `ROW_NUMBER()`: asigna un número único secuencial por partición.

- `RANK()`: igual que ROW_NUMBER pero con empates; deja huecos.

- `DENSE_RANK()`: igual que RANK pero sin huecos.

- `NTILE(n)`: divide las filas en n grupos aproximadamente iguales.

Ejemplo:

```sql
SELECT nombre, salario,
       ROW_NUMBER() OVER (ORDER BY salario DESC) AS row_num,
       RANK()       OVER (ORDER BY salario DESC) AS rank,
       DENSE_RANK() OVER (ORDER BY salario DESC) AS dense_rank,
       NTILE(4)     OVER (ORDER BY salario DESC) AS cuartil
FROM empleados;
```

## Funciones de Valor

- `LAG(col, n)`: valor de la columna n filas antes.

- `LEAD(col, n)`: valor n filas después.

- `FIRST_VALOR(col)`: primer valor en la ventana.

- `LAST_VALUE(col)`: último valor.

- `NTH_VALUE(col, n)`: enésimo valor.

Muy útiles para comparar filas consecutivas.

```sql
SELECT fecha, monto,
       LAG(monto, 1) OVER (ORDER BY fecha) AS monto_anterior,
       monto - LAG(monto, 1) OVER (ORDER BY fecha) AS diferencia
FROM ventas_diarias;
```

## Funciones de Agregación como Ventana

Las funciones como SUM, AVG pueden usarse como ventana para obtener totales acumulados, promedios móviles, etc.

```sql
SELECT fecha, monto,
       SUM(monto) OVER (ORDER BY fecha) AS acumulado,
       AVG(monto) OVER (ORDER BY fecha ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS media_movil_3
FROM ventas_diarias;
```

## Definiendo Marcos de Ventana

El marco especifica qué filas incluir. Opciones:

- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (desde el inicio hasta la actual).

- `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` (anterior, actual y siguiente).

- `RANGE` trabaja con valores en lugar de filas (útil para fechas).

## Ejemplo: Ranking por Departamento

Obtener los empleados mejor pagados de cada departamento (top 3).

```sql
WITH ranked AS (
    SELECT nombre, salario, departamento_id,
           ROW_NUMBER() OVER (PARTITION BY departamento_id ORDER BY salario DESC) AS rn
    FROM empleados
)
SELECT * FROM ranked WHERE rn <= 3;
```

# Limpieza y Transformación de Datos

La preparación de datos (data cleaning) es una de las tareas que más tiempo consume en proyectos de datos. SQL ofrece potentes funciones para limpiar y transformar.

## Manejo de Valores Nulos

- `COALESCE(expr1, expr2, ...)`: devuelve el primer no nulo.

- `NULLIF(expr1, expr2)`: devuelve NULL si expr1 = expr2, sino expr1.

- `IS NULL / IS NOT NULL`: para filtrar.

Ejemplo: Reemplazar nulos en email por 'sin email'.

```sql
SELECT nombre, COALESCE(email, 'sin email') AS email
FROM clientes;
```

## Funciones de Texto

- `UPPER(texto)`, `LOWER(texto)`, `INITCAP(texto)`: cambiar mayúsculas/minúsculas.

- `CONCAT(str1, str2)` o `||` (operador de concatenación).

- `SUBSTRING(texto FROM inicio FOR longitud)` o `SUBSTR`.

- `REPLACE(texto, de, a)`: reemplazar subcadenas.

- `TRANSLATE(texto, de, a)`: reemplazo carácter a carácter.

- `TRIM([LEADING|TRAILING|BOTH] caracter FROM texto)`: eliminar espacios o caracteres.

- `LPAD(texto, longitud, relleno)`, `RPAD`: rellenar a la izquierda/derecha.

Ejemplo: Normalizar nombres.

```sql
SELECT UPPER(TRIM(nombre)) AS nombre_normalizado
FROM clientes;
```

### Expresiones Regulares

Muchos motores soportan expresiones regulares (PostgreSQL: ` `, `! `, `REGEXP_MATCH`, `REGEXP_REPLACE`, `REGEXP_SPLIT`). Ejemplo: Extraer el dominio de un email.

```sql
SELECT email, SUBSTRING(email FROM '@(.*)$') AS dominio
FROM clientes;
-- O con regexp_replace
SELECT REGEXP_REPLACE(email, '.*@', '') AS dominio FROM clientes;
```

## Funciones de Fecha y Hora

- `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `NOW()`.

- `EXTRACT(campo FROM fecha)`: extrae año, mes, día, hora, etc.

- `DATE_PART(’campo’, fecha)`: similar a EXTRACT.

- `DATE_TRUNC(’unidad’, fecha)`: trunca a la unidad especificada (año, mes, día).

- `fecha1 - fecha2`: diferencia en días (depende del motor).

- `INTERVAL`: suma o resta intervalos.

Ejemplo: Obtener ventas del último mes.

```sql
SELECT * FROM ventas
WHERE fecha >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
  AND fecha < DATE_TRUNC('month', CURRENT_DATE);
```

## Conversión de Tipos

- `CAST(expr AS tipo)` o `expr::tipo` (PostgreSQL).

- `TO_CHAR(fecha, formato)`: formatear fecha a texto.

- `TO_DATE(texto, formato)`: texto a fecha.

- `TO_NUMBER(texto, formato)`: texto a número.

Ejemplo: Convertir texto '2025-03-01' a fecha.

```sql
SELECT TO_DATE('2025-03-01', 'YYYY-MM-DD');
```

## Ejercicio Resuelto: Limpieza de Datos de Clientes

**Enunciado:** La tabla `clientes_raw` tiene columnas: id, nombre (con espacios y mayúsculas inconsistentes), email (con algunos nulos), fecha_registro (como texto en formato 'DD/MM/YYYY'). Limpiar:

1.  Nombre en formato título (initcap) y sin espacios extras.

2.  Email: si es nulo, asignar 'pendiente@mail.com'.

3.  Fecha convertida a tipo DATE.

**Solución:**

```sql
SELECT id,
       INITCAP(TRIM(nombre)) AS nombre_limpio,
       COALESCE(email, 'pendiente@mail.com') AS email,
       TO_DATE(fecha_registro, 'DD/MM/YYYY') AS fecha_registro_date
FROM clientes_raw;
```

# Técnicas Avanzadas para ETL y Transformación

En procesos ETL (Extract, Transform, Load) y preparación de datos para IA, se requieren técnicas más sofisticadas.

## Tablas Temporales

Las tablas temporales existen solo durante la sesión. Útiles para almacenar resultados intermedios.

```sql
CREATE TEMP TABLE resumen_departamento AS
SELECT departamento_id, AVG(salario) AS salario_prom
FROM empleados
GROUP BY departamento_id;
```

Luego podemos usarla en consultas posteriores.

## Common Table Expressions (CTEs)

Las CTEs (WITH) mejoran la legibilidad y permiten recursión. Ya las vimos en la sesión anterior, pero aquí las aplicamos a transformaciones complejas. Ejemplo: Encontrar empleados con salario superior al promedio de su departamento.

```sql
WITH promedio_depto AS (
    SELECT departamento_id, AVG(salario) AS salario_prom
    FROM empleados
    GROUP BY departamento_id
)
SELECT e.nombre, e.salario, e.departamento_id, pd.salario_prom
FROM empleados e
JOIN promedio_depto pd ON e.departamento_id = pd.departamento_id
WHERE e.salario > pd.salario_prom;
```

## Subconsultas Correlacionadas en Transformaciones

Son útiles para comparar cada fila con un agregado de su grupo, como ya vimos.

## Condicionales con CASE

`CASE` permite lógica condicional en SQL. Es muy usado para categorizar datos.

```sql
SELECT nombre, salario,
       CASE
           WHEN salario < 2000 THEN 'Bajo'
           WHEN salario < 4000 THEN 'Medio'
           ELSE 'Alto'
       END AS categoria_salarial
FROM empleados;
```

## Pivotes con CASE y Agregación Condicional

Para convertir filas en columnas (pivot), se puede usar CASE dentro de funciones de agregación. Ejemplo: Mostrar ventas por año y mes en columnas separadas.

```sql
SELECT producto,
       SUM(CASE WHEN EXTRACT(month FROM fecha) = 1 THEN monto ELSE 0 END) AS enero,
       SUM(CASE WHEN EXTRACT(month FROM fecha) = 2 THEN monto ELSE 0 END) AS febrero,
       ...
FROM ventas
GROUP BY producto;
```

También se puede usar `FILTER (WHERE condición)` en algunos motores:

```sql
SELECT producto,
       SUM(monto) FILTER (WHERE EXTRACT(month FROM fecha) = 1) AS enero,
       SUM(monto) FILTER (WHERE EXTRACT(month FROM fecha) = 2) AS febrero
FROM ventas
GROUP BY producto;
```

## Uniones y Combinaciones Avanzadas

A veces necesitamos combinar datos de diferentes fuentes con UNION, INTERSECT, EXCEPT.

```sql
-- Clientes que son también proveedores
SELECT id, nombre FROM clientes
INTERSECT
SELECT id, nombre FROM proveedores;
```

## Manejo de Duplicados

Podemos eliminar duplicados usando DISTINCT, o identificar duplicados con ROW_NUMBER.

```sql
WITH duplicados AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
    FROM clientes
)
DELETE FROM duplicados WHERE rn > 1; -- En PostgreSQL se necesita usar la tabla real
```

Pero en la práctica, para eliminar, se usa una subconsulta o una tabla temporal.

## Ejercicio Resuelto: Transformación Compleja

**Enunciado:** Dada una tabla `ventas` (id, fecha, producto, monto) y `productos` (id, nombre, categoria), se pide:

1.  Crear una tabla temporal con las ventas del último trimestre.

2.  Calcular el total de ventas por categoría y mes, mostrando los meses como columnas (pivot).

3.  Añadir una columna que clasifique las ventas como 'baja' (\<100), 'media' (100-500), 'alta' (\>500).

4.  Finalmente, mostrar solo las categorías con ventas totales \> 1000.

**Solución:**

```sql
-- 1. Tabla temporal con ventas último trimestre
CREATE TEMP TABLE ventas_trimestre AS
SELECT v.*, p.categoria
FROM ventas v
JOIN productos p ON v.producto = p.id
WHERE v.fecha >= DATE_TRUNC('quarter', CURRENT_DATE) - INTERVAL '3 months';

-- 2. Pivot por mes
SELECT categoria,
       SUM(CASE WHEN EXTRACT(month FROM fecha) = 1 THEN monto ELSE 0 END) AS enero,
       SUM(CASE WHEN EXTRACT(month FROM fecha) = 2 THEN monto ELSE 0 END) AS febrero,
       SUM(CASE WHEN EXTRACT(month FROM fecha) = 3 THEN monto ELSE 0 END) AS marzo,
       SUM(monto) AS total
FROM ventas_trimestre
GROUP BY categoria;

-- 3. Clasificación
SELECT *,
       CASE
           WHEN monto < 100 THEN 'baja'
           WHEN monto <= 500 THEN 'media'
           ELSE 'alta'
       END AS clasificacion
FROM ventas_trimestre;

-- 4. Filtro por total > 1000
SELECT categoria, SUM(monto) AS total
FROM ventas_trimestre
GROUP BY categoria
HAVING SUM(monto) > 1000;
```

# Ejercicios Resueltos Adicionales

## Ejercicio 1: Series Temporales

**Enunciado:** Calcular la media móvil de 7 días de las ventas diarias. **Solución:**

```sql
SELECT fecha, monto,
       AVG(monto) OVER (ORDER BY fecha ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS media_movil_7
FROM ventas_diarias;
```

## Ejercicio 2: Limpieza de Texto y Expresiones Regulares

**Enunciado:** Extraer el código postal de una dirección que está en el formato \"Calle Falsa 123, 28080 Madrid\". Se supone que el código postal son 5 dígitos. **Solución:**

```sql
SELECT direccion,
       SUBSTRING(direccion FROM '[0-9]{5}') AS codigo_postal
FROM clientes;
```

## Ejercicio 3: Agregación Condicional

**Enunciado:** Contar empleados por departamento y también cuántos tienen salario \> 3000. **Solución:**

```sql
SELECT departamento_id,
       COUNT(*) AS total,
       COUNT(*) FILTER (WHERE salario > 3000) AS altos_salarios
FROM empleados
GROUP BY departamento_id;
```

## Ejercicio 4: CTE Recursiva para Jerarquía

**Enunciado:** Obtener todos los subordinados de un empleado dado (id=5) en una tabla empleados con jefe_id. **Solución:**

```sql
WITH RECURSIVE subordinados AS (
    SELECT id, nombre, jefe_id
    FROM empleados
    WHERE id = 5
    UNION ALL
    SELECT e.id, e.nombre, e.jefe_id
    FROM empleados e
    JOIN subordinados s ON e.jefe_id = s.id
)
SELECT * FROM subordinados;
```

# Glosario de Términos

Función de agregación

:   Función que opera sobre un conjunto de filas y devuelve un único valor (SUM, AVG, COUNT).

GROUP BY

:   Agrupa filas que comparten valores en columnas especificadas.

HAVING

:   Filtra grupos después de la agregación.

ROLLUP

:   Extensión de GROUP BY que genera subtotales.

CUBE

:   Genera todas las combinaciones de subtotales.

Función de ventana

:   Función que calcula un valor sobre un conjunto de filas relacionadas sin colapsar.

OVER

:   Define la ventana para funciones de ventana.

PARTITION BY

:   Divide el conjunto en particiones dentro de una ventana.

ROW_NUMBER

:   Asigna un número secuencial único.

RANK

:   Asigna rango con huecos.

DENSE_RANK

:   Asigna rango sin huecos.

LAG / LEAD

:   Accede a filas anteriores o posteriores.

COALESCE

:   Devuelve el primer valor no nulo.

NULLIF

:   Devuelve NULL si dos expresiones son iguales.

Expresión regular

:   Patrón para búsqueda y manipulación de texto.

CTE

:   Common Table Expression (WITH clause).

Tabla temporal

:   Tabla que existe solo durante una sesión.

CASE

:   Expresión condicional.

Pivot

:   Transformación de filas en columnas.

