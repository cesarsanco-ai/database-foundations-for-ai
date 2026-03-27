
## Sesión 2
# MODELAMIENTO DE DATOS
## De la realidad del negocio al esquema físico

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026


---

## Introducción al Modelamiento de Datos

El modelamiento de datos es el proceso de diseñar la estructura que almacenará la información de manera eficiente, sin redundancias y reflejando fielmente las reglas del negocio. Es la fase más creativa y abstracta, pero también la que determina el éxito o fracaso de un sistema de información. En esta sesión, exploraremos desde los conceptos fundamentales hasta las técnicas avanzadas, pasando por ejemplos prácticos y ejercicios resueltos que te ayudarán a dominar esta disciplina esencial.

### ¿Por qué es importante modelar?

Imagina construir un edificio sin planos arquitectónicos. El resultado sería caótico, inseguro e imposible de mantener. Lo mismo ocurre con las bases de datos: sin un buen modelo, los datos se corrompen, las consultas son lentas y el sistema se vuelve inmanejable.

El modelamiento de datos nos permite:
- **Comunicar** las necesidades de información entre usuarios de negocio y técnicos.
- **Documentar** la estructura de los datos para futuros mantenimientos.
- **Optimizar** el rendimiento mediante un diseño adecuado.
- **Garantizar** la integridad y consistencia de los datos.

---

## El Ciclo de Modelamiento: Del Concepto al Disco

El diseño de una base de datos sigue un proceso de abstracción en tres niveles, cada uno con un propósito y un público diferente. Este enfoque garantiza que la estructura final soporte las reglas de negocio y sea implementable en el SGBD elegido.

```
[Modelo Conceptual]  -->  [Modelo Lógico]  -->  [Modelo Físico]
(Entidad-Relación)       (Tablas, PK, FK)     (Tipos, Índices, Disco)
     Traducción              Implementación
```

### Modelo Entidad-Relación (E-R) - Nivel Conceptual

Es un modelo de alto nivel, independiente del software de base de datos, que representa la realidad del negocio mediante **entidades**, **atributos** y **relaciones**.

#### Entidades
Una entidad es un objeto del mundo real distinguible de otros por sus características. Se representa con un rectángulo y su nombre en singular.
- **Entidad fuerte**: existe por sí misma (ej. `Cliente`).
- **Entidad débil**: depende de otra entidad para existir (ej. `Pago` depende de `Factura`). Se representa con rectángulo de doble línea.

#### Atributos
Los atributos son las propiedades que describen a las entidades. Se clasifican en:
- **Simples** (atómicos): no se pueden dividir (ej. `edad`).
- **Compuestos**: se pueden dividir en subpartes (ej. `dirección` → `calle, ciudad, CP`).
- **Multivalorados**: pueden tener varios valores (ej. `teléfonos` de un cliente). Se representan con doble óvalo.
- **Derivados**: se calculan a partir de otros (ej. `edad` a partir de `fecha_nacimiento`). Se representan con óvalo punteado.
- **Clave**: atributo o conjunto de atributos que identifica de forma única a cada instancia de la entidad. Se subraya.

#### Relaciones
Una relación asocia dos o más entidades. Se representa con un rombo y un verbo. La **cardinalidad** indica el número de instancias de una entidad que pueden relacionarse con otra. Las cardinalidades más comunes son:
- **1:1** (uno a uno): Un empleado tiene un único carné de estacionamiento, y viceversa.
- **1:N** (uno a muchos): Un cliente puede realizar muchos pedidos, pero un pedido pertenece a un solo cliente.
- **N:M** (muchos a muchos): Un estudiante puede cursar muchas asignaturas, y una asignatura puede ser cursada por muchos estudiantes.

Además, la **participación** puede ser:
- **Total** (obligatoria): toda instancia de la entidad debe participar en la relación (línea continua).
- **Parcial** (opcional): no todas las instancias participan (línea discontinua).

#### Ejemplo gráfico
```
[Cliente] --1--> (Realiza) --N--> [Pedido]
```

### Modelo Lógico - Nivel de Implementación Independiente

En este nivel, el modelo E-R se traduce a un esquema de tablas, columnas y restricciones, sin detalles específicos del SGBD.

#### Reglas de transformación
1. Cada entidad fuerte se convierte en una tabla. Sus atributos simples se convierten en columnas. Los atributos compuestos se descomponen en columnas simples. Los atributos multivalorados requieren una tabla aparte.
2. La clave primaria (PK) de la tabla es el atributo clave de la entidad.
3. Para relaciones 1:N, se añade la PK de la entidad del lado "1" como clave foránea (FK) en la tabla de la entidad del lado "N".
4. Para relaciones N:M, se crea una tabla intermedia que contiene las PK de ambas entidades como FK, formando una clave primaria compuesta (o una nueva clave simple).
5. Las entidades débiles se convierten en tablas con una FK hacia la entidad fuerte, y su clave incluye la FK más un discriminador.

#### Normalización
La normalización es un proceso que aplica reglas para eliminar redundancias y evitar anomalías en las inserciones, actualizaciones y eliminaciones.

- **Primera Forma Normal (1FN):** Los atributos deben ser atómicos (no repetir grupos). Por ejemplo, en lugar de una columna "teléfonos" con varios números, crear una tabla separada `TelefonosCliente`.
- **Segunda Forma Normal (2FN):** Cumplir 1FN y todo atributo no clave debe depender completamente de la clave primaria (aplicable en tablas con claves compuestas). Por ejemplo, en una tabla `DetallePedido` con PK (id_pedido, id_producto), el atributo "precio" depende del producto, no del pedido; debe separarse.
- **Tercera Forma Normal (3FN):** Cumplir 2FN y ningún atributo no clave debe depender transitivamente de otro atributo no clave. Por ejemplo, en una tabla `Empleado` con columnas (id_empleado, nombre, id_departamento, nombre_departamento), "nombre_departamento" depende de "id_departamento", no directamente de la clave. Debe separarse en tabla `Departamento`.
- **Forma Normal de Boyce-Codd (BCNF):** Una versión más estricta de 3FN donde toda dependencia funcional no trivial es una superclave.
- **Cuarta Forma Normal (4FN):** Trata dependencias multivaluadas. Por ejemplo, si un empleado tiene varios títulos y varios idiomas, se crean tablas separadas para evitar redundancia.
- **Quinta Forma Normal (5FN):** Relacionada con dependencias de proyección-join. En la práctica, hasta 3FN/BCNF es suficiente para la mayoría de los sistemas.

### Modelo Físico - Nivel de Implementación Real

Es la implementación concreta en el SGBD elegido. Se definen tipos de datos específicos, índices, particionamiento y otros detalles.

#### Tipos de datos comunes
- `INT`, `BIGINT`, `SMALLINT` para enteros.
- `VARCHAR(n)` para cadenas de longitud variable.
- `CHAR(n)` para cadenas fijas.
- `TEXT` para textos largos.
- `DATE`, `TIME`, `TIMESTAMP` para fechas y horas.
- `DECIMAL(p,s)` o `NUMERIC(p,s)` para valores exactos con decimales (moneda).
- `FLOAT`, `DOUBLE` para aproximaciones.
- `BOOLEAN` para verdadero/falso.
- `BLOB` para datos binarios (imágenes, archivos).
- Tipos especiales: `JSON`, `JSONB` (PostgreSQL), `GEOMETRY` (espacial).


---

## Ejemplos Prácticos y Ejercicios Resueltos

### Caso 1: Sistema Universitario

**Requisitos:** Se necesita almacenar información de estudiantes, profesores, asignaturas y las matrículas (estudiantes se matriculan de asignaturas, con nota y fecha). Los profesores imparten asignaturas. Cada asignatura pertenece a un departamento.

#### Modelo Conceptual
Entidades: Estudiante, Profesor, Asignatura, Departamento.
Relaciones:
- Matricula (N:M entre Estudiante y Asignatura, con atributos nota y fecha).
- Imparte (N:M entre Profesor y Asignatura, con atributo semestre).
- Pertenece (N:1 entre Asignatura y Departamento).

#### Modelo Lógico
- Estudiante (id_est: INT, nombre: VARCHAR, email: VARCHAR, fecha_nac: DATE)
- Profesor (id_prof: INT, nombre: VARCHAR, email: VARCHAR, id_depto: INT) -- FK a Departamento
- Departamento (id_depto: INT, nombre: VARCHAR)
- Asignatura (id_asig: INT, nombre: VARCHAR, creditos: INT, id_depto: INT FK)
- Matricula (id_est, id_asig, semestre: VARCHAR, nota: DECIMAL, fecha_mat: DATE)
- Imparte (id_prof, id_asig, semestre: VARCHAR)

#### Modelo Físico en PostgreSQL
```sql
CREATE TABLE departamento (
    id_depto SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE estudiante (
    id_est SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    fecha_nac DATE
);

CREATE TABLE profesor (
    id_prof SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    id_depto INT REFERENCES departamento(id_depto)
);

CREATE TABLE asignatura (
    id_asig SERIAL PRIMARY KEY,
    nombre VARCHAR(200) NOT NULL,
    creditos INT CHECK (creditos > 0),
    id_depto INT NOT NULL REFERENCES departamento(id_depto)
);

CREATE TABLE matricula (
    id_est INT REFERENCES estudiante(id_est),
    id_asig INT REFERENCES asignatura(id_asig),
    semestre VARCHAR(10) NOT NULL,
    nota DECIMAL(3,1) CHECK (nota BETWEEN 0 AND 10),
    fecha_mat DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (id_est, id_asig, semestre)
);

CREATE TABLE imparte (
    id_prof INT REFERENCES profesor(id_prof),
    id_asig INT REFERENCES asignatura(id_asig),
    semestre VARCHAR(10) NOT NULL,
    PRIMARY KEY (id_prof, id_asig, semestre)
);

CREATE INDEX idx_matricula_est ON matricula(id_est);
CREATE INDEX idx_imparte_asig ON imparte(id_asig);
```

### Caso 2: Sistema Hospitalario

**Requisitos:** Registrar pacientes, médicos, ingresos, diagnósticos y tratamientos. Un paciente puede tener varios ingresos. Cada ingreso tiene un diagnóstico principal y varios tratamientos. Los médicos atienden ingresos.

#### Modelo Conceptual
Entidades: Paciente, Medico, Ingreso, Diagnostico, Tratamiento.
Relaciones:
- Tiene (1:N Paciente-Ingreso)
- Atiende (N:M Medico-Ingreso, con atributo rol)
- Asigna (N:1 Ingreso-Diagnostico)
- Recibe (N:M Ingreso-Tratamiento, con fecha y dosis)

#### Modelo Lógico
- Paciente (id_pac: INT, nombre, fecha_nac, nss)
- Medico (id_med: INT, nombre, especialidad)
- Diagnostico (id_diag: INT, descripcion, cie10)
- Tratamiento (id_trat: INT, nombre, descripcion)
- Ingreso (id_ingreso: INT, fecha_ing, fecha_alta, id_pac FK, id_diag FK)
- Atiende (id_ingreso, id_med, rol: VARCHAR)
- Recibe (id_ingreso, id_trat, fecha: DATE, dosis: VARCHAR)

#### Modelo Físico en PostgreSQL
```sql
CREATE TABLE paciente (
    id_pac SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    fecha_nac DATE,
    nss VARCHAR(20) UNIQUE
);

CREATE TABLE medico (
    id_med SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    especialidad VARCHAR(100)
);

CREATE TABLE diagnostico (
    id_diag SERIAL PRIMARY KEY,
    descripcion TEXT,
    cie10 VARCHAR(10) UNIQUE
);

CREATE TABLE tratamiento (
    id_trat SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT
);

CREATE TABLE ingreso (
    id_ingreso SERIAL PRIMARY KEY,
    fecha_ing DATE NOT NULL,
    fecha_alta DATE,
    id_pac INT NOT NULL REFERENCES paciente(id_pac),
    id_diag INT REFERENCES diagnostico(id_diag)
);

CREATE TABLE atiende (
    id_ingreso INT REFERENCES ingreso(id_ingreso),
    id_med INT REFERENCES medico(id_med),
    rol VARCHAR(50),
    PRIMARY KEY (id_ingreso, id_med)
);

CREATE TABLE recibe (
    id_ingreso INT REFERENCES ingreso(id_ingreso),
    id_trat INT REFERENCES tratamiento(id_trat),
    fecha DATE NOT NULL,
    dosis VARCHAR(50),
    PRIMARY KEY (id_ingreso, id_trat, fecha)
);
```

---

## Ejercicios Propuestos con Solución Paso a Paso

### Ejercicio 1: Biblioteca

**Enunciado:** Diseñar el modelo conceptual, lógico y físico para una biblioteca que gestiona libros, autores, socios y préstamos. Un libro puede tener varios autores. Un socio puede tomar prestados varios libros, y un libro puede ser prestado varias veces a lo largo del tiempo (registrar fecha de préstamo y devolución).

#### Paso 1: Modelo Conceptual
Identificamos las entidades:
- **Libro**: atributos: ISBN, título, año, editorial.
- **Autor**: atributos: id_autor, nombre, nacionalidad.
- **Socio**: atributos: id_socio, nombre, dirección, teléfono.
- **Préstamo**: atributos: fecha_préstamo, fecha_devolución (puede ser nula si aún no se ha devuelto).

Relaciones:
- **Escribe** (N:M entre Autor y Libro). No tiene atributos.
- **Realiza** (1:N entre Socio y Préstamo). Un socio puede tener varios préstamos, pero un préstamo pertenece a un solo socio.
- **Incluye** (N:M entre Préstamo y Libro). Un préstamo puede incluir varios libros, y un libro puede estar en varios préstamos a lo largo del tiempo.

#### Paso 2: Modelo Lógico (con N:M)
- Autor (id_autor: INT, nombre: VARCHAR, nacionalidad: VARCHAR)
- Libro (isbn: VARCHAR(20), titulo: VARCHAR(200), anio: INT, editorial: VARCHAR(100))
- Socio (id_socio: INT, nombre: VARCHAR(100), direccion: VARCHAR(200), telefono: VARCHAR(20))
- Prestamo (id_prestamo: INT, fecha_prestamo: DATE, fecha_devolucion: DATE, id_socio: INT FK)
- LibroAutor (id_autor, isbn) -- tabla intermedia para N:M
- PrestamoLibro (id_prestamo, isbn) -- tabla intermedia para N:M

#### Paso 3: Modelo Físico en PostgreSQL
```sql
CREATE TABLE autor (
    id_autor SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    nacionalidad VARCHAR(50)
);

CREATE TABLE libro (
    isbn VARCHAR(20) PRIMARY KEY,
    titulo VARCHAR(200) NOT NULL,
    anio INT,
    editorial VARCHAR(100)
);

CREATE TABLE socio (
    id_socio SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    direccion VARCHAR(200),
    telefono VARCHAR(20)
);

CREATE TABLE prestamo (
    id_prestamo SERIAL PRIMARY KEY,
    fecha_prestamo DATE NOT NULL DEFAULT CURRENT_DATE,
    fecha_devolucion DATE,
    id_socio INT NOT NULL REFERENCES socio(id_socio)
);

CREATE TABLE libro_autor (
    id_autor INT REFERENCES autor(id_autor),
    isbn VARCHAR(20) REFERENCES libro(isbn),
    PRIMARY KEY (id_autor, isbn)
);

CREATE TABLE prestamo_libro (
    id_prestamo INT REFERENCES prestamo(id_prestamo),
    isbn VARCHAR(20) REFERENCES libro(isbn),
    PRIMARY KEY (id_prestamo, isbn)
);

-- Índices para rendimiento
CREATE INDEX idx_prestamo_socio ON prestamo(id_socio);
CREATE INDEX idx_libro_autor_isbn ON libro_autor(isbn);
```

### Ejercicio 2: Red Social

**Enunciado:** Modelar una red social donde hay usuarios, publicaciones, comentarios y likes. Los usuarios pueden seguir a otros usuarios. Las publicaciones pueden tener múltiples comentarios y likes. Los comentarios también pueden tener likes.

#### Paso 1: Modelo Conceptual
Entidades:
- Usuario: id, nombre, email, fecha_registro.
- Publicacion: id, contenido, fecha, id_usuario (autor).
- Comentario: id, contenido, fecha, id_usuario (autor), id_publicacion (a la que pertenece).
- Like: puede ser a publicación o a comentario. Decidimos modelar dos entidades de like separadas o una generalización. Optaremos por dos tablas: LikePublicacion y LikeComentario.

Relaciones:
- Sigue (N:M entre Usuario y Usuario): un usuario puede seguir a muchos y ser seguido por muchos.
- Publica (1:N Usuario-Publicacion).
- Comenta (1:N Usuario-Comentario; 1:N Publicacion-Comentario).
- LikePublicacion (N:M entre Usuario y Publicacion).
- LikeComentario (N:M entre Usuario y Comentario).

#### Paso 2: Modelo Lógico
- Usuario (id_usuario, nombre, email, fecha_reg)
- Publicacion (id_pub, contenido, fecha, id_usuario FK)
- Comentario (id_com, contenido, fecha, id_usuario FK, id_pub FK)
- Sigue (id_seguidor, id_seguido) -- ambos FK a Usuario
- LikePublicacion (id_usuario, id_pub, fecha_like)
- LikeComentario (id_usuario, id_com, fecha_like)

#### Paso 3: Modelo Físico en PostgreSQL
```sql
CREATE TABLE usuario (
    id_usuario SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    fecha_reg DATE DEFAULT CURRENT_DATE
);

CREATE TABLE publicacion (
    id_pub SERIAL PRIMARY KEY,
    contenido TEXT NOT NULL,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    id_usuario INT NOT NULL REFERENCES usuario(id_usuario)
);

CREATE TABLE comentario (
    id_com SERIAL PRIMARY KEY,
    contenido TEXT NOT NULL,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    id_usuario INT NOT NULL REFERENCES usuario(id_usuario),
    id_pub INT NOT NULL REFERENCES publicacion(id_pub)
);

CREATE TABLE sigue (
    id_seguidor INT REFERENCES usuario(id_usuario),
    id_seguido INT REFERENCES usuario(id_usuario),
    PRIMARY KEY (id_seguidor, id_seguido)
);

CREATE TABLE like_publicacion (
    id_usuario INT REFERENCES usuario(id_usuario),
    id_pub INT REFERENCES publicacion(id_pub),
    fecha_like TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id_usuario, id_pub)
);

CREATE TABLE like_comentario (
    id_usuario INT REFERENCES usuario(id_usuario),
    id_com INT REFERENCES comentario(id_com),
    fecha_like TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id_usuario, id_com)
);

-- Índices para búsquedas comunes
CREATE INDEX idx_publicacion_usuario ON publicacion(id_usuario);
CREATE INDEX idx_comentario_publicacion ON comentario(id_pub);
```

### Ejercicio 3: Normalización

**Enunciado:** Dada la siguiente tabla no normalizada, aplícale 1FN, 2FN y 3FN.

```
Pedido(id_pedido, fecha, cliente, telefono_cliente, direccion_cliente,
       (id_producto, nombre_producto, cantidad, precio_unitario))
```

Donde la parte entre paréntesis representa una repetición de grupos (varios productos por pedido).

#### Paso 1: Primera Forma Normal (1FN)
Eliminar grupos repetidos creando una tabla separada para los detalles del pedido.

Tabla `Pedido` original:
- id_pedido (PK)
- fecha
- cliente
- telefono_cliente
- direccion_cliente

Tabla `DetallePedido`:
- id_pedido (FK)
- id_producto
- nombre_producto
- cantidad
- precio_unitario
- PK: (id_pedido, id_producto)

#### Paso 2: Segunda Forma Normal (2FN)
Analizamos dependencias funcionales en `DetallePedido`. La clave es (id_pedido, id_producto). Los atributos no clave son: nombre_producto, cantidad, precio_unitario.
- cantidad depende de la clave completa (id_pedido, id_producto): OK.
- precio_unitario podría depender solo de id_producto (el precio de un producto no varía por pedido). Esto viola 2FN.
- nombre_producto también depende solo de id_producto.

Para 2FN, creamos una tabla `Producto`:
- Producto (id_producto, nombre_producto, precio_unitario)

Y en `DetallePedido` dejamos solo id_producto (FK), cantidad, y el precio podría ser el del momento (si se quiere histórico, se mantiene precio_unitario en detalle).

Tablas resultantes:
- Pedido (id_pedido, fecha, cliente, telefono_cliente, direccion_cliente)
- Producto (id_producto, nombre_producto, precio_unitario)
- DetallePedido (id_pedido, id_producto, cantidad)

#### Paso 3: Tercera Forma Normal (3FN)
En la tabla `Pedido`, observamos dependencias transitivas:
- cliente → telefono_cliente, direccion_cliente. El teléfono y dirección dependen del cliente, no directamente de id_pedido.

Para 3FN, separamos `Cliente`:
- Cliente (id_cliente, nombre, telefono, direccion)

Y en `Pedido` ponemos id_cliente (FK).

Modelo final normalizado:
- Cliente (id_cliente, nombre, telefono, direccion)
- Producto (id_producto, nombre_producto, precio_unitario)
- Pedido (id_pedido, fecha, id_cliente FK)
- DetallePedido (id_pedido, id_producto, cantidad)

---

## Conceptos Fundamentales para el Día a Día

### Propiedades ACID
Garantizan transacciones confiables.
- **Atomicidad:** Una transacción se ejecuta completamente o no se ejecuta (todo o nada).
- **Consistencia:** La transacción lleva la base de datos de un estado válido a otro, respetando restricciones.
- **Aislamiento (Isolation):** Las transacciones concurrentes no interfieren entre sí. Niveles: Read Uncommitted, Read Committed, Repeatable Read, Serializable.
- **Durabilidad:** Una vez confirmada, la transacción persiste aunque el sistema falle.

### Niveles de Aislamiento
1. **Read Uncommitted:** Puede leer datos no confirmados (dirty reads).
2. **Read Committed:** Solo lee datos confirmados. Evita dirty reads, pero pueden ocurrir non-repeatable reads.
3. **Repeatable Read:** Asegura que si se lee una fila dos veces en la misma transacción, el valor no cambia. Puede tener phantom reads.
4. **Serializable:** Máximo aislamiento, las transacciones se ejecutan como si fueran secuenciales.

### Bloqueos (Locks)
Mecanismos para controlar la concurrencia.
- **Compartidos (Shared):** Para lectura. Varios pueden leer.
- **Exclusivos (Exclusive):** Para escritura. Solo uno.
- Pueden causar *deadlocks* (bloqueos mutuos), que el SGBD detecta y resuelve.

---



---

