# Sesión 3 – Normalización e Integridad en SQL

**Autor:** Carlos César Sánchez Coronel  

---

## 1. Introducción: ¿Por qué normalizar?

El diseño de una base de datos relacional debe garantizar que la información sea **consistente**, **no redundante** y **fácil de mantener**. Una estructura mal diseñada puede provocar:

- **Anomalías de inserción**: No se pueden añadir ciertos datos sin tener otros presentes.
- **Anomalías de actualización**: Un mismo dato aparece en múltiples lugares; al modificarlo debe actualizarse en todos ellos, riesgo de inconsistencia.
- **Anomalías de eliminación**: Al borrar una entidad se pierde información relacionada que debería conservarse.

La **normalización** es el proceso de organizar los atributos en tablas para eliminar estas anomalías. Se basa en el concepto de **dependencias funcionales** y aplica una serie de reglas (formas normales) que progresivamente reducen la redundancia y mejoran la integridad.

---

## 2. Dependencias Funcionales

Una **dependencia funcional** (DF) es una relación entre dos conjuntos de atributos. Significa que, para cualquier par de tuplas, si coinciden en todos los atributos de \(X\), también coinciden en todos los atributos de \(Y\). En otras palabras, \(X\) determina funcionalmente a \(Y\).

- **Ejemplo**: En una tabla `Empleado(id_empleado, nombre, id_departamento, nombre_departamento)`, se cumple:
  - \(id\_empleado \rightarrow nombre, id\_departamento, nombre\_departamento\) (la clave determina todos los demás).
  - \(id\_departamento \rightarrow nombre\_departamento\) (el código del departamento determina su nombre). Esta última es una dependencia transitiva si \(id\_departamento\) no es clave.

Las dependencias funcionales son la base para identificar las formas normales.

---

## 3. Formas Normales

### 3.1 Primera Forma Normal (1NF)

**Definición**: Una tabla está en 1NF si todos los atributos son **atómicos** (no contienen grupos repetidos ni valores compuestos).

- **Violación**: Tener una columna que almacena múltiples valores separados por comas, o una columna con una lista de teléfonos.
- **Solución**: Crear una tabla separada para el conjunto multivalorado, con una clave foránea hacia la tabla principal.

**Ejemplo**:  
Tabla original: `Cliente(id, nombre, telefonos)` con telefonos = "123,456,789".  
1NF: Crear `Cliente(id, nombre)` y `Telefono(id_cliente, telefono)`.

---

### 3.2 Segunda Forma Normal (2NF)

**Definición**: Una tabla está en 2NF si está en 1NF y todos los atributos **no clave** dependen completamente de la **clave primaria completa** (no de una parte de la clave). Esta condición solo es relevante cuando la clave primaria es **compuesta**.

- **Violación**: En una tabla `DetallePedido(id_pedido, id_producto, cantidad, precio_unitario)`, con clave compuesta `(id_pedido, id_producto)`, el atributo `precio_unitario` depende solo de `id_producto` (no de la clave completa).  
- **Solución**: Separar en `Producto(id_producto, precio_unitario)` y `DetallePedido(id_pedido, id_producto, cantidad)`.

---

### 3.3 Tercera Forma Normal (3NF)

**Definición**: Una tabla está en 3NF si está en 2NF y ningún atributo **no clave** depende transitivamente de la clave primaria. Es decir, no debe haber dependencias funcionales de la forma \(X \rightarrow Y\) donde \(X\) no es superclave e \(Y\) es un atributo no clave.

- **Violación**: En `Empleado(id_empleado, nombre, id_departamento, nombre_departamento)`, la dependencia transitiva es:  
  \(id\_empleado \rightarrow id\_departamento\) y \(id\_departamento \rightarrow nombre\_departamento\). Por tanto, `nombre_departamento` depende transitivamente de la clave.
- **Solución**: Crear una tabla `Departamento(id_departamento, nombre_departamento)` y mantener solo `id_departamento` en `Empleado`.

---

### 3.4 Forma Normal de Boyce‑Codd (BCNF)

**Definición**: Una tabla está en BCNF si, para cada dependencia funcional no trivial \(X \rightarrow Y\), \(X\) es una **superclave** (contiene una clave candidata). BCNF es más estricta que 3NF; toda tabla en BCNF también está en 3NF, pero no al revés.

- **Ejemplo**: Supongamos una tabla `ProfesorAsignatura(id_profesor, id_asignatura, departamento)` con las siguientes dependencias funcionales:
  - `id_profesor → departamento` (cada profesor pertenece a un departamento).
  - `id_asignatura → departamento` (cada asignatura pertenece a un departamento).
  - La clave primaria es `(id_profesor, id_asignatura)`.
  - Ambas dependencias tienen determinantes que no son superclave, por lo que la tabla viola BCNF aunque esté en 3NF. La solución es descomponer en tres tablas: `Profesor(id_profesor, departamento)`, `Asignatura(id_asignatura, departamento)`, y `ProfesorAsignatura(id_profesor, id_asignatura)`.

---

### 3.5 Formas Normales Superiores (4NF, 5NF)

Estas formas normales abordan dependencias multivaluadas y de proyección‑join, comunes en escenarios de normalización avanzada.

- **4NF**: Elimina las dependencias multivaluadas no triviales. Ejemplo: una tabla `Empleado(id, idiomas, titulos)` con dos atributos multivalorados independientes viola 4NF. Se descompone en `EmpleadoIdioma(id, idioma)` y `EmpleadoTitulo(id, titulo)`.
- **5NF**: Trata dependencias de join; es muy teórica y rara vez necesaria en la práctica.

En la mayoría de los sistemas, alcanzar **3NF o BCNF** es suficiente para eliminar las anomalías más comunes.

---

## 4. Integridad de Datos en SQL

La integridad de datos garantiza que la base de datos refleja correctamente la realidad del negocio y no contiene errores. SQL proporciona varios mecanismos para imponer restricciones a nivel de dominio, entidad y referencias.

### 4.1 Integridad de Dominio

Define los valores válidos para una columna.

- **NOT NULL**: Impide valores nulos.
- **CHECK**: Verifica una condición booleana. Ejemplo: `CHECK (creditos > 0)`, `CHECK (modalidad IN ('texto','audio','imagen','video'))`.
- **Tipos de datos**: Seleccionar el tipo adecuado (`INT`, `DATE`, `VARCHAR(100)`) ya restringe el dominio.

### 4.2 Integridad de Entidad (Clave Primaria)

Asegura que cada fila sea única y no nula en el atributo (o conjunto de atributos) que constituye la clave primaria.

- **PRIMARY KEY**: Define la clave primaria, garantizando unicidad y no nulidad.
- **UNIQUE**: Asegura unicidad, pero permite valores nulos (aunque en la práctica muchos SGBD permiten solo un nulo por columna con UNIQUE).

### 4.3 Integridad Referencial (Clave Foránea)

Garantiza que los valores de una columna (o conjunto) corresponden a valores existentes en otra tabla (normalmente su clave primaria). Mantiene la coherencia entre tablas relacionadas.

- **FOREIGN KEY**: Crea un vínculo referencial. Opciones de acción para `ON DELETE` y `ON UPDATE`:
  - **CASCADE**: Propaga la eliminación/actualización a las filas dependientes.
  - **SET NULL**: Establece el valor de la FK a NULL.
  - **SET DEFAULT**: Asigna el valor por defecto.
  - **RESTRICT / NO ACTION**: Impide la operación si hay dependencias.

### 4.4 Restricciones Definidas por el Usuario (CHECK, UNIQUE, NOT NULL)

Además de las anteriores, se pueden crear restricciones complejas con **CHECK** (expresiones SQL) y **UNIQUE** (claves alternativas). También se pueden combinar en una misma tabla.

### 4.5 Triggers y Reglas de Negocio

Cuando las restricciones declarativas no son suficientes (por ejemplo, para validar datos entre múltiples tablas o ejecutar acciones automáticas), se usan **triggers** (disparadores). Un trigger es un bloque de código que se ejecuta automáticamente antes o después de una operación (INSERT, UPDATE, DELETE). Ejemplo: Un trigger que actualiza el campo `ultima_actividad` de una sesión cada vez que se inserta un mensaje.

---

## 5. Normalización vs. Desnormalización: El Trade‑Off

La normalización reduce redundancia y mejora la integridad, pero puede provocar un mayor número de JOINs en las consultas, lo que impacta en el rendimiento. En ciertos escenarios, se opta por **desnormalizar** parcialmente:

- **Data warehouses**: Se usan esquemas en estrella (star schema) con tablas de hechos (desnormalizadas) para consultas agregadas rápidas.
- **Sistemas OLTP (transaccionales)**: Normalización hasta 3NF es habitual para garantizar consistencia y facilitar actualizaciones.
- **Índices y optimizadores**: Un buen diseño físico (índices, particionamiento) puede mitigar la penalización de los JOINs.

La decisión de normalizar o desnormalizar depende del patrón de uso (más lecturas o más escrituras), la criticidad de la consistencia y la escalabilidad requerida.

---

## 6. Ejemplo Práctico Integrado

Vamos a aplicar la normalización y las restricciones de integridad sobre una tabla inicial que viola múltiples formas normales.

**Tabla original (no normalizada)**:

```
Pedido (
    id_pedido,
    fecha,
    cliente,
    telefono_cliente,
    direccion_cliente,
    productos (lista de id_producto, nombre, precio, cantidad)
)
```

### Paso 1: 1NF – Eliminar grupos repetidos

Separar la lista de productos en una tabla `DetallePedido`. Obtenemos:

```
Pedido (id_pedido, fecha, cliente, telefono_cliente, direccion_cliente)
DetallePedido (id_pedido, id_producto, nombre_producto, precio_unitario, cantidad)
```

### Paso 2: 2NF – Dependencias parciales

En `DetallePedido`, la clave es `(id_pedido, id_producto)`. Los atributos `nombre_producto` y `precio_unitario` dependen solo de `id_producto`. Los separamos en una tabla `Producto`:

```
Producto (id_producto, nombre_producto, precio_unitario)
DetallePedido (id_pedido, id_producto, cantidad)
```

### Paso 3: 3NF – Dependencias transitivas

En `Pedido`, los atributos `telefono_cliente` y `direccion_cliente` dependen de `cliente`. Por tanto, creamos una tabla `Cliente`:

```
Cliente (id_cliente, nombre, telefono, direccion)
Pedido (id_pedido, fecha, id_cliente)
```

Ahora todas las tablas están en 3NF. Añadimos restricciones de integridad:

```sql
CREATE TABLE cliente (
    id_cliente SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(20),
    direccion TEXT
);

CREATE TABLE producto (
    id_producto SERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    precio_unitario DECIMAL(10,2) NOT NULL CHECK (precio_unitario > 0)
);

CREATE TABLE pedido (
    id_pedido SERIAL PRIMARY KEY,
    fecha DATE NOT NULL DEFAULT CURRENT_DATE,
    id_cliente INT NOT NULL REFERENCES cliente(id_cliente)
);

CREATE TABLE detalle_pedido (
    id_pedido INT REFERENCES pedido(id_pedido) ON DELETE CASCADE,
    id_producto INT REFERENCES producto(id_producto),
    cantidad INT NOT NULL CHECK (cantidad > 0),
    PRIMARY KEY (id_pedido, id_producto)
);
```

---

## 7. Resumen y Buenas Prácticas

- **Normaliza hasta 3NF** en la mayoría de los sistemas transaccionales.
- **Identifica dependencias funcionales** para detectar violaciones.
- **Usa restricciones declarativas** (`PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `UNIQUE`) siempre que sea posible; son más eficientes y seguras que los triggers.
- **Documenta el diseño** con diagramas y descripciones de las dependencias.
- **Considera la desnormalización** solo cuando haya un problema de rendimiento demostrado con perfiles de carga reales.
- **Evalúa el impacto en escrituras** antes de añadir índices (que también afectan al rendimiento, aunque no son parte de la normalización).

La combinación de un buen diseño normalizado y el uso adecuado de las restricciones de integridad es la base para construir sistemas de datos confiables, mantenibles y escalables.