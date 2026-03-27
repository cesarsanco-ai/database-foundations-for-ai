# Laboratorio 1: Modelamiento de Datos

## Aplicado al Caso Aletza

---

### Objetivo

Aplicar los conceptos de modelamiento de datos (conceptual, lógico, físico) y normalización al caso de estudio **Aletza**, complementando con ejercicios prácticos que consoliden la capacidad de diseñar esquemas eficientes y libres de redundancias. Al finalizar, el estudiante será capaz de:

- Construir un diagrama Entidad-Relación a partir de requisitos.
- Transformar un modelo conceptual en un esquema lógico normalizado.
- Implementar el modelo físico en PostgreSQL (Supabase) con restricciones e índices avanzados.
- Evaluar las ventajas de diferentes estrategias de indexación.
- Reflexionar sobre el modelamiento en entornos NoSQL y Big Data.

---

## 1. Introducción Teórica Breve

El modelamiento de datos es el proceso de diseñar la estructura que almacenará la información de manera eficiente, sin redundancias y reflejando las reglas del negocio. Se desarrolla en tres niveles de abstracción:

| Nivel | Propósito | Artefacto |
|-------|-----------|-----------|
| **Conceptual** | Representar la realidad del negocio de forma independiente del SGBD. | Diagrama Entidad-Relación (E-R) con entidades, atributos, relaciones y cardinalidades. |
| **Lógico** | Traducir el modelo conceptual a tablas, columnas y claves (PK, FK), aplicando normalización para eliminar redundancias. | Esquema de tablas normalizadas, con dependencias funcionales resueltas. |
| **Físico** | Implementar el modelo en un SGBD concreto, definiendo tipos de datos, índices, particionamiento, etc. | Scripts SQL `CREATE TABLE`, índices, restricciones de integridad. |

**Normalización** es el proceso de organizar los atributos para evitar anomalías de actualización. Las formas normales más utilizadas son:

- **1FN**: Atributos atómicos; eliminar grupos repetitivos.
- **2FN**: Atributos no clave dependen completamente de la clave primaria (en tablas con clave compuesta).
- **3FN**: Eliminar dependencias transitivas (atributos no clave dependen de otros atributos no clave).

**Índices** aceleran las consultas a costa de escrituras más lentas. Los tipos comunes son B-Tree (por defecto), Hash, GiST, GIN, BRIN.

---

## 2. Actividades

### Actividad 1: Modelo Conceptual de Aletza (10 min)

**Requisitos del caso Aletza (resumidos):**

- Un **usuario** se registra proporcionando username y email.
- Durante el registro, el usuario envía 10 muestras de voz diciendo la palabra mágica `"aletza"`. Estas muestras se almacenan como un **perfil de voz** (vector biométrico).
- Para usar el asistente, el usuario se autentica con un mensaje de voz que dice `"aletza"`. Si coincide, se crea una **sesión** con duración máxima de 1 hora de inactividad.
- Durante la sesión, el usuario puede enviar **mensajes** de diferentes modalidades (texto, audio, imagen, video). Cada mensaje es procesado por módulos de IA que dejan trazabilidad en **logs de ejecución**.

**Tarea:** Dibuja (o describe textualmente) un diagrama Entidad-Relación que incluya las entidades, sus atributos, las relaciones y las cardinalidades. Identifica al menos una entidad débil si la hubiera.

---

### Actividad 2: Normalización de Logs de Ejecución (15 min)

Actualmente, la tabla `logs_ejecucion` se diseñó con los campos:

```
logs_ejecucion (
    id SERIAL PRIMARY KEY,
    mensaje_id INT,
    modulo TEXT,
    tiempo_ejecucion_ms INT,
    exito BOOLEAN,
    error TEXT,
    ejecutado_en TIMESTAMPTZ
)
```

Se observa que cada vez que se procesa un mensaje se insertan varios registros (uno por módulo). Analiza si esta tabla cumple con las formas normales.

**a)** ¿Hay algún atributo multivalorado o repetitivo que viole 1FN? Justifica.

**b)** Supongamos que añadimos a `logs_ejecucion` el campo `nombre_modulo` (que describe el módulo, p.ej. "Clasificador NLP") y `version_modulo` (versión del módulo). ¿Qué dependencias funcionales aparecen? ¿Cómo se resolvería en 2FN y 3FN? Propón un esquema normalizado.

**c)** ¿Qué ventajas e inconvenientes tendría normalizar más esta tabla para un sistema que requiere alta velocidad de escritura (muchos logs por segundo)?

---

### Actividad 3: Transformación a Modelo Lógico y Físico (20 min)

Partiendo del modelo conceptual que diseñaste en la Actividad 1, genera el modelo lógico (tablas, claves primarias, foráneas) y luego escribe los comandos SQL para crear las tablas en Supabase (PostgreSQL). Asegúrate de incluir:

- Tipos de datos adecuados (por ejemplo, `JSONB` para vectores de voz y metadatos de IA).
- Restricciones `CHECK` para campos como `modalidad`.
- Claves foráneas con `ON DELETE CASCADE` donde corresponda.
- Al menos un índice B-Tree sobre una columna que se consulte frecuentemente.

**Pista:** El modelo original del caso de estudio (proporcionado en la sesión 1) ya tiene una versión inicial. Puedes mejorarlo si detectas oportunidades de normalización o restricciones adicionales.

---

### Actividad 4: Reflexión sobre Modelamiento NoSQL (10 min)

Aletza es un sistema multimodal que maneja datos de naturaleza variada: transaccionales (usuarios, sesiones), semiestructurados (metadatos IA) y no estructurados (audios, imágenes).

**a)** ¿Qué tipo de base de datos NoSQL (clave-valor, documental, columnar, grafos) sería más adecuada para almacenar los **logs de ejecución** si se necesitara escalar horizontalmente y analizar grandes volúmenes con agregaciones? Explica por qué.

**b)** Si decidiéramos usar MongoDB para almacenar los mensajes y sus metadatos, ¿cómo modelarías la relación entre un mensaje y sus logs de ejecución? ¿Documentos anidados o referencias? Argumenta.

**c)** ¿Qué aspectos del modelo relacional actual se beneficiarían de un enfoque híbrido (SQL + NoSQL)? Menciona al menos dos.

---

## 3. Solucionario

### Actividad 1: Modelo Conceptual de Aletza

**Descripción textual del diagrama E-R:**

- **Entidad `Usuario`** (fuerte)  
  Atributos: `id_usuario`, `username`, `email`, `creado_en`.

- **Entidad `PerfilVoz`** (débil, dependiente de `Usuario`)  
  Atributos: `id_perfil`, `palabra_magica`, `vector_voz` (multivalorado o compuesto), `muestras_tomadas`, `estado`.  
  Relación: **`tiene_perfil`** (1:1 o 1:N? Un usuario tiene un solo perfil de voz completo, aunque podrían almacenarse versiones. Por simplicidad, 1:1).

- **Entidad `Sesion`** (fuerte, pero con dependencia temporal)  
  Atributos: `id_sesion`, `token`, `canal`, `iniciada_en`, `ultima_actividad`, `expira_en`.  
  Relación: **`inicia`** (1:N entre `Usuario` y `Sesion`). Un usuario puede tener varias sesiones (activas o pasadas).

- **Entidad `Mensaje`**  
  Atributos: `id_mensaje`, `contenido`, `modalidad`, `metadata_ia`, `enviado_en`.  
  Relación: **`pertenece`** (1:N entre `Sesion` y `Mensaje`).  
  Relación: **`envia`** (1:N entre `Usuario` y `Mensaje`). Nota: aunque el remitente es el usuario, está implícito por la sesión.

- **Entidad `LogEjecucion`**  
  Atributos: `id_log`, `modulo`, `tiempo_ejecucion_ms`, `exito`, `error`, `ejecutado_en`.  
  Relación: **`genera`** (1:N entre `Mensaje` y `LogEjecucion`). Cada mensaje genera varios logs (uno por módulo).

**Cardinalidades principales:**
- `Usuario` --- (1) : (N) --- `Sesion`
- `Usuario` --- (1) : (1) --- `PerfilVoz`
- `Sesion` --- (1) : (N) --- `Mensaje`
- `Mensaje` --- (1) : (N) --- `LogEjecucion`

**Entidad débil:** `PerfilVoz` depende de `Usuario`; su existencia no tiene sentido sin un usuario.

---

### Actividad 2: Normalización de Logs de Ejecución

**a)** La tabla `logs_ejecucion` actual cumple 1FN porque cada columna tiene valores atómicos y no hay grupos repetidos. Cada fila representa un módulo ejecutado para un mensaje.

**b)** Si añadimos `nombre_modulo` y `version_modulo`, aparecen dependencias funcionales:

- `modulo` → `nombre_modulo` (el código del módulo determina su nombre)
- `modulo` → `version_modulo` (cada módulo tiene una versión)

Esto viola 2FN (atributos no clave dependen de parte de la clave, pero en este caso `modulo` no es clave primaria; sin embargo, la tabla tiene su propia clave `id`). En realidad, estamos introduciendo una dependencia transitiva si consideramos que `nombre_modulo` y `version_modulo` dependen de `modulo`, no de `id`. Para 3FN deberíamos separar:

```
modulo (
    codigo TEXT PRIMARY KEY,
    nombre TEXT NOT NULL,
    version TEXT
)

logs_ejecucion (
    id SERIAL PRIMARY KEY,
    mensaje_id INT,
    modulo_codigo TEXT REFERENCES modulo(codigo),
    tiempo_ejecucion_ms INT,
    exito BOOLEAN,
    error TEXT,
    ejecutado_en TIMESTAMPTZ
)
```

**c)** Ventajas de normalizar: evita redundancia, facilita cambios de nombre/versión (un solo lugar). Inconvenientes: requiere JOIN para obtener el nombre del módulo, lo que añade latencia. En un sistema de alta escritura, los JOIN pueden ser costosos; a menudo se prefiere desnormalizar para rendimiento, asumiendo que la información del módulo cambia raramente.

---

### Actividad 3: Modelo Lógico y Físico

**Modelo lógico (tablas):**

```
Usuario (id_usuario, username, email, creado_en)
PerfilVoz (id_perfil, usuario_id, palabra_magica, vector_voz, muestras_tomadas, estado, creado_en)
Sesion (id_sesion, usuario_id, token, canal, iniciada_en, ultima_actividad, expira_en)
Mensaje (id_mensaje, sesion_id, remitente_id, contenido, modalidad, metadata_ia, enviado_en)
LogEjecucion (id_log, mensaje_id, modulo, tiempo_ejecucion_ms, exito, error, ejecutado_en)
```

**Modelo físico en PostgreSQL (Supabase):**

```sql
-- Usuario
CREATE TABLE usuario (
    id_usuario SERIAL PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- PerfilVoz
CREATE TABLE perfil_voz (
    id_perfil SERIAL PRIMARY KEY,
    usuario_id INT NOT NULL REFERENCES usuario(id_usuario) ON DELETE CASCADE,
    palabra_magica TEXT NOT NULL DEFAULT 'aletza',
    vector_voz JSONB,
    muestras_tomadas INT DEFAULT 0 CHECK (muestras_tomadas BETWEEN 0 AND 10),
    estado TEXT DEFAULT 'incompleto' CHECK (estado IN ('incompleto', 'completo')),
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Sesion
CREATE TABLE sesion (
    id_sesion SERIAL PRIMARY KEY,
    usuario_id INT NOT NULL REFERENCES usuario(id_usuario) ON DELETE CASCADE,
    token TEXT UNIQUE NOT NULL,
    canal TEXT CHECK (canal IN ('telegram', 'web', 'app')),
    iniciada_en TIMESTAMPTZ DEFAULT NOW(),
    ultima_actividad TIMESTAMPTZ DEFAULT NOW(),
    expira_en TIMESTAMPTZ DEFAULT NOW() + INTERVAL '1 hour'
);

-- Mensaje
CREATE TABLE mensaje (
    id_mensaje SERIAL PRIMARY KEY,
    sesion_id INT NOT NULL REFERENCES sesion(id_sesion) ON DELETE CASCADE,
    remitente_id INT NOT NULL REFERENCES usuario(id_usuario),
    contenido TEXT,
    modalidad TEXT CHECK (modalidad IN ('texto', 'audio', 'imagen', 'video')),
    metadata_ia JSONB,
    enviado_en TIMESTAMPTZ DEFAULT NOW()
);

-- LogEjecucion
CREATE TABLE log_ejecucion (
    id_log SERIAL PRIMARY KEY,
    mensaje_id INT NOT NULL REFERENCES mensaje(id_mensaje) ON DELETE CASCADE,
    modulo TEXT NOT NULL,
    tiempo_ejecucion_ms INT,
    exito BOOLEAN,
    error TEXT,
    ejecutado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Índices sugeridos
CREATE INDEX idx_sesion_usuario ON sesion(usuario_id);
CREATE INDEX idx_mensaje_sesion ON mensaje(sesion_id);
CREATE INDEX idx_mensaje_remitente ON mensaje(remitente_id);
CREATE INDEX idx_log_mensaje ON log_ejecucion(mensaje_id);
```


---

### Actividad 4: Reflexión sobre Modelamiento NoSQL

**a)** Para logs de ejecución con necesidad de escalabilidad horizontal y análisis agregado, una base de datos **columnar** (como Apache Cassandra o ScyllaDB) es adecuada porque permite escribir grandes volúmenes con baja latencia y consultas agregadas eficientes si se modelan las particiones correctamente. Otra alternativa es una base de datos **de series temporales** (TimescaleDB, InfluxDB) si los logs se consultan principalmente por tiempo.

**b)** En MongoDB, se podría modelar la relación **mensaje–logs** de dos formas:

- **Anidación**: incluir un array de logs dentro del documento del mensaje. Esto es eficiente si los logs se consultan siempre junto con el mensaje y el número de logs por mensaje es limitado (ej. menos de 100). La escritura es atómica y no requiere JOIN.
- **Referencias**: tener una colección separada `logs` con un campo `mensaje_id`. Esto permite crecer ilimitadamente y consultar logs sin traer el mensaje, pero requiere dos consultas o un `$lookup` (similar a JOIN).

Para un sistema de alta escritura, la anidación puede ser problemática si el array crece demasiado; se recomienda **referencias** para escalar.

**c)** Beneficios de un enfoque híbrido (SQL + NoSQL) en Aletza:

1. **Mensajes y metadatos IA** podrían almacenarse en MongoDB como documentos, aprovechando la flexibilidad de esquema para añadir nuevos campos sin migraciones.
2. **Logs de ejecución** podrían ir a una base de datos de series temporales (TimescaleDB) o columnar (Cassandra) para retener grandes volúmenes y facilitar análisis agregados, manteniendo los datos transaccionales (usuarios, sesiones) en PostgreSQL para integridad referencial y ACID.



---

# 🔐 PROPIEDADES ACID (explicadas con Aletza)

## 🧨 1. Atomicidad (todo o nada)

La atomicidad significa que una transacción **se ejecuta completamente o no se ejecuta en absoluto**.

👉 **Ejemplo en Aletza:**
Cuando un usuario se registra, ocurren varias cosas:

* Se crea el usuario
* Se crea su perfil de voz

Si por algún error falla la creación del perfil de voz, **no debería quedar el usuario creado a medias**.

✔ Entonces:

* O se guardan ambos (usuario + perfil)
* O no se guarda ninguno

👉 Esto evita datos incompletos o inconsistentes.

---

## 🧩 2. Consistencia (respeta reglas)

La consistencia asegura que la base de datos **siempre cumple las reglas definidas** (restricciones, claves, formatos).

👉 **Ejemplo en Aletza:**
En la tabla `Mensaje`, el campo `modalidad` solo puede ser:

* texto
* audio
* imagen
* video

Si alguien intenta guardar:

> modalidad = "pdf"

❌ La base de datos lo rechaza

✔ Porque rompería las reglas del sistema.

👉 Entonces, la BD pasa de un estado válido a otro estado válido.

---

## 🔄 3. Aislamiento (transacciones independientes)

El aislamiento significa que **varias transacciones pueden ocurrir al mismo tiempo sin interferirse**.

👉 **Ejemplo en Aletza:**
Un mismo usuario puede tener:

* una sesión en web
* otra en el celular

Ambas pueden:

* enviar mensajes
* actualizar actividad

✔ El sistema maneja esto sin mezclar datos ni generar errores.

👉 Es como si cada transacción ocurriera “sola”, aunque haya muchas al mismo tiempo.

---

## 💾 4. Durabilidad (no se pierde)

La durabilidad significa que **una vez que los datos se guardan (COMMIT), no se pierden**, incluso si el sistema falla.

👉 **Ejemplo en Aletza:**
Un usuario envía un mensaje:

> "hola"

El sistema lo guarda.

Luego el servidor se cae 😬

✔ Cuando vuelve a funcionar:

* El mensaje sigue ahí

👉 Porque la base de datos garantiza que los datos confirmados son permanentes.

---

# 🔒 NIVELES DE AISLAMIENTO (explicados simple)

---

## 🔴 1. Read Uncommitted

Permite leer datos **que aún no están confirmados**.

👉 Ejemplo:
Una transacción cambia el email de un usuario pero no guarda aún.

Otra transacción lo lee.

❌ Problema:

* Puede leer algo que luego se deshace

👉 Esto se llama *dirty read*

---

## 🟡 2. Read Committed (el que usa PostgreSQL / Supabase)

Solo puedes leer datos **ya confirmados**.

👉 Ejemplo en Aletza:

* Un mensaje se está editando pero no se ha guardado
* Otro usuario intenta verlo

✔ No verá el cambio hasta que se confirme

👉 Evita datos “sucios”

---

## 🟠 3. Repeatable Read

Si lees un dato, **no cambiará dentro de la misma transacción**.

👉 Ejemplo:
Lees un mensaje:

> "hola"

Otro proceso lo cambia a:

> "hola mundo"

✔ Tú seguirás viendo "hola" dentro de tu transacción

👉 Evita inconsistencias al volver a leer

---

## 🟢 4. Serializable

Es el nivel más alto.

👉 Hace que las transacciones se comporten como si ocurrieran **una por una**.

👉 Ejemplo en Aletza:
Dos procesos intentan crear una sesión al mismo tiempo para el mismo usuario.

✔ El sistema permite solo una correctamente
❌ La otra falla

👉 Evita errores complejos

---

# 🔐 BLOQUEOS (LOCKS)

Son mecanismos para evitar conflictos cuando varios acceden a los mismos datos.

---

## 📖 Bloqueo Compartido (Shared)

👉 Para lectura

✔ Muchos pueden leer al mismo tiempo

Ejemplo:

* varios usuarios viendo mensajes

---

## ✍️ Bloqueo Exclusivo (Exclusive)

👉 Para escritura

✔ Solo uno puede modificar

Ejemplo:

* actualizar contenido de un mensaje

❌ Nadie más puede modificar ese mismo mensaje al mismo tiempo

---

## ⚠️ Deadlock (bloqueo mutuo)

Ocurre cuando dos transacciones **se bloquean entre sí**.

👉 Ejemplo en Aletza:

* Transacción A bloquea `Usuario` y quiere `Sesion`
* Transacción B bloquea `Sesion` y quiere `Usuario`

💥 Ninguna puede avanzar

✔ El sistema detecta esto y cancela una

---

# 🏁 RESUMEN FINAL 

* **Atomicidad:** registro completo o nada
* **Consistencia:** se respetan reglas (ej: modalidad)
* **Aislamiento:** múltiples sesiones sin interferencia
* **Durabilidad:** los datos no se pierden


