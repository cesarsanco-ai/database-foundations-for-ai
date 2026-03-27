# Laboratorio 2: El Universo del Dato – Fundamentos y Soporte Físico

## Aplicado al Caso Aletza

---

### Objetivo
Comprender los conceptos fundamentales de **dato, información, conocimiento, metadatos, hardware, gobernanza y roles** en la gestión de datos, aplicándolos directamente al caso de estudio **Aletza**. Al finalizar, el estudiante será capaz de:
- Clasificar distintos tipos de datos en un sistema real.
- Identificar metadatos y su utilidad.
- Evaluar implicaciones de hardware y complejidad computacional.
- Distinguir roles profesionales en un equipo de datos.
- Proponer medidas de gobernanza y ética.

---

## 1. Introducción teórica: del dato al conocimiento

En el corazón de cualquier sistema de bases de datos se encuentra la información. Esta no surge de la nada; es el resultado de un proceso de refinamiento que parte de elementos más básicos.

### 1.1 Dato, información, conocimiento

| Concepto | Definición | Ejemplo en Aletza |
|----------|------------|-------------------|
| **Dato** | Unidad mínima, hecho objetivo crudo. | `"aletza"` (audio), `"2025-03-26 10:00:00"`, `{"embedding": [0.12,0.34,0.56]}` |
| **Información** | Dato procesado y contextualizado. | Saber que el usuario **Juan** dijo *aletza* el **26/03/2025** y se autenticó. |
| **Conocimiento** | Información asimilada que permite acción o predicción. | Detectar que los usuarios piden más soporte después de las 18:00 y ajustar turnos. |

### 1.2 Clasificación de los datos según su estructura

- **Estructurados**: siguen un esquema fijo (filas y columnas).  
  *Ejemplo*: `usuarios.id`, `usuarios.username`.
- **Semiestructurados**: poseen organización pero no esquema fijo (JSON, XML).  
  *Ejemplo*: `perfiles_voz.vector_voz` (JSON con el embedding biométrico).
- **No estructurados**: carecen de organización predefinida.  
  *Ejemplo*: archivos de audio, transcripciones de texto libre.

### 1.3 Metadatos: los datos que hablan de los datos

- **Descriptivos**: ayudan a descubrir e identificar.  
  *Ejemplo*: `modalidad` en `mensajes` (indica si es texto, audio, imagen, video).
- **Estructurales**: describen la organización interna.  
  *Ejemplo*: definición de columnas en una tabla (nombre, tipo de dato, longitud).
- **Administrativos**: información técnica y de gestión.  
  *Ejemplo*: `creado_en`, `expira_en`, permisos, políticas de retención.

### 1.4 Hardware y complejidad computacional

- **Jerarquía de memoria** (latencia típica):
  - Registros CPU: < 1 ns
  - Caché L1/L2: 1–10 ns
  - RAM (buffer pool): ~100 ns
  - SSD/NVMe: 20–200 μs (microsegundos)
  - HDD: 4–10 ms (milisegundos)
  - Red/SAN: 0.5–2 ms adicionales

- **Complejidad de búsqueda**:
  - *Table scan*: O(n) – recorre todas las filas.
  - *Índice B-Tree*: O(log n) – búsqueda por igualdad o rango.
  - *Índice hash*: O(1) promedio – solo búsquedas exactas.

### 1.5 Roles en la gestión de datos

| Rol | Responsabilidad principal en Aletza |
|-----|-------------------------------------|
| **DBA** | Administrar la base de datos: copias de seguridad, monitoreo, ajuste de índices, alta disponibilidad. |
| **Data Engineer** | Construir pipelines de datos (ETL) para procesar logs, integrar con data warehouse, mantener la infraestructura de datos. |
| **Data Analyst** | Crear informes y dashboards sobre el uso del asistente (métricas de sesiones, mensajes, tiempo de respuesta). |
| **Data Scientist** | Entrenar modelos de autenticación por voz, mejorar los módulos de IA (clasificación de intención, generación de respuestas). |
| **Data Steward** | Definir políticas de calidad, privacidad y retención de datos; asegurar cumplimiento normativo. |

### 1.6 Gobernanza y ética

- **Privacidad**: Los datos biométricos (voz) requieren protección especial.  
- **Cumplimiento normativo**: GDPR, CCPA exigen consentimiento, derecho al olvido, minimización de datos.  
- **Sesgo algorítmico**: Modelos entrenados con datos históricos pueden perpetuar discriminación.  
- **Transparencia**: Es fundamental auditar el linaje de los datos para explicar decisiones automáticas.

---

## 2. Actividades

### Actividad 1: Clasificación de datos en Aletza (10 min)

Clasifica los siguientes elementos de Aletza según su tipo (estructurado, semiestructurado, no estructurado) y justifica tu respuesta.

| Elemento | Tipo | Justificación |
|----------|------|---------------|
| a) El campo `username` en la tabla `usuarios` | | |
| b) El archivo de audio enviado por el usuario | | |
| c) El campo `metadata_ia` que contiene JSON con la intención detectada | | |
| d) La lista de vectores de voz almacenados en `perfiles_voz.vector_voz` (JSONB) | | |
| e) El log de ejecución con campos `modulo`, `tiempo_ejecucion_ms`, `exito` | | |

---

### Actividad 2: Identificando metadatos (10 min)

Revisa las tablas del caso Aletza (usuarios, perfiles_voz, sesiones, mensajes, logs_ejecucion). Proporciona un ejemplo concreto de cada tipo de metadato:

- **Metadato descriptivo**:  
- **Metadato estructural**:  
- **Metadato administrativo**:  

Además, explica cómo los metadatos pueden ayudar a un **data analyst** a entender qué datos están disponibles para un informe.

---

### Actividad 3: Hardware y complejidad (15 min)

**a) Jerarquía de memoria**  
Supón que Aletza tiene una tabla `mensajes` con 10 millones de registros. Si se ejecuta una consulta para listar los mensajes de un usuario específico `remitente_id = 5`, ¿qué nivel de memoria (RAM, SSD, HDD) influirá más en el rendimiento si no hay índice? ¿Y si existe un índice B-Tree sobre `remitente_id`? Explica la diferencia en latencia.

**b) Impacto de índices en escrituras**  
¿Qué efecto tienen los índices en las operaciones de INSERT, UPDATE y DELETE? ¿Cómo equilibrarías la decisión de crear índices en una tabla que recibe muchas escrituras pero también muchas consultas?

---

### Actividad 4: Asignación de roles (10 min)

El equipo de Aletza está compuesto por cinco personas: Ana, Luis, Marta, Carlos y Elena. Lee las siguientes tareas y asigna a cada persona el rol más adecuado (DBA, Data Engineer, Data Analyst, Data Scientist, Data Steward). Explica brevemente por qué.

| Persona | Tarea | Rol | Justificación |
|---------|-------|-----|---------------|
| Ana | Diseña y entrena el modelo de reconocimiento de voz para autenticación. | | |
| Luis | Configura la replicación de la base de datos entre dos regiones para alta disponibilidad. | | |
| Marta | Construye un pipeline que cada hora extrae los logs de ejecución, los transforma y los carga en un data warehouse. | | |
| Carlos | Crea un dashboard con métricas de uso diario (número de sesiones, mensajes por canal). | | |
| Elena | Define la política de retención de datos: los mensajes se guardan 90 días, los perfiles de voz 5 años después de la última sesión. | | |

---

### Actividad 5: Gobernanza y ética (15 min)

**a) Privacidad**  
¿Qué medidas implementarías en Aletza para proteger los perfiles de voz (datos biométricos) y cumplir con el GDPR? Menciona al menos tres.

**b) Derecho al olvido**  
Un usuario solicita eliminar su cuenta y todos sus datos. ¿Cómo debe responder el sistema? ¿Qué relaciones entre tablas pueden facilitar esta operación?

**c) Riesgos éticos**  
La autenticación en Aletza es únicamente por voz, sin alternativa. ¿Qué riesgos éticos presenta esta decisión? Propón una solución que mantenga la seguridad pero sea inclusiva.

---

### Actividad 6: Primer contacto con Supabase (opcional, 10 min)

Si dispones de acceso a Internet y a Supabase, sigue estos pasos:

1. Crea un proyecto en [supabase.com](https://supabase.com).
2. En el **SQL Editor**, ejecuta el siguiente script para crear las tablas básicas de Aletza. No te preocupes por entender cada línea; en próximos laboratorios profundizaremos.

```sql
-- Copia y pega el script del caso de estudio (tablas: usuarios, perfiles_voz, sesiones, mensajes, logs_ejecucion)
-- (El script está disponible en la sección 1.1 del documento del caso de estudio)
```

3. Una vez creadas las tablas, explora las tablas del sistema ejecutando:

```sql
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';
```

4. Observa que además de nuestras tablas aparecen otras (migraciones, etc.). **Estas son metadatos del sistema**.

**Pregunta**: ¿Por qué es importante conocer la estructura de las tablas (metadatos estructurales) antes de escribir consultas?

---

## 3. Solucionario

### Actividad 1: Clasificación de datos en Aletza

| Elemento | Tipo | Justificación |
|----------|------|---------------|
| a) `username` | Estructurado | Es una columna de texto con longitud fija en una tabla relacional, sigue un esquema rígido. |
| b) Archivo de audio | No estructurado | No tiene formato predefinido; es un flujo binario que requiere procesamiento especial (transcripción). |
| c) `metadata_ia` (JSON) | Semiestructurado | Contiene campos anidados (intención, sentimiento) que pueden variar según el mensaje, pero está organizado como JSON. |
| d) Vectores de voz (JSONB) | Semiestructurado | Aunque guarda arrays numéricos, el formato JSON permite flexibilidad (ej. diferentes dimensiones de embedding). |
| e) Log de ejecución | Estructurado | Cada campo tiene un tipo fijo (modulo:text, tiempo:integer, exito:boolean) y sigue un esquema rígido. |

---

### Actividad 2: Identificando metadatos

- **Metadato descriptivo**: `modalidad` en la tabla `mensajes` indica si el mensaje es texto, audio, imagen o video. Ayuda a entender qué tipo de contenido se maneja.
- **Metadato estructural**: En `information_schema.columns`, la definición de la columna `expira_en` como `TIMESTAMPTZ` y `DEFAULT NOW() + INTERVAL '1 hour'`. Define cómo se almacena la fecha de expiración.
- **Metadato administrativo**: `creado_en` en cualquier tabla, que registra la fecha de creación del registro. También `expira_en` en `sesiones` indica la política de caducidad.

**Ayuda al data analyst**: Los metadatos estructurales le permiten conocer qué tablas existen, qué columnas tienen y sus tipos, para poder formular consultas correctas. Los metadatos descriptivos (como `modalidad`) le indican el significado de los valores.

---

### Actividad 3: Hardware y complejidad

**a) Jerarquía de memoria**
- **Sin índice**: La base de datos debe realizar un *table scan* (lectura secuencial) de toda la tabla `mensajes` para encontrar las filas con `remitente_id = 5`. Esto implica leer millones de registros desde disco (HDD o SSD) hacia la RAM. Si la tabla está en HDD, la latencia será alta (varios segundos). Si está en SSD, será más rápida pero aún así el tiempo será proporcional al tamaño de la tabla.
- **Con índice B-Tree**: El índice permite localizar directamente las filas con `remitente_id = 5` en O(log n). Solo se accede a los bloques que contienen esas filas, que pueden residir en RAM (buffer pool) si ya están cacheadas, reduciendo drásticamente la E/S.

**b) Impacto de índices en escrituras**
- Cada INSERT, UPDATE o DELETE requiere actualizar el índice además de los datos. Esto añade sobrecarga de CPU y E/S.
- Equilibrio: En tablas con alta relación lectura/escritura, conviene crear índices para las consultas más críticas. Si las escrituras son muy frecuentes, se deben elegir índices con cuidado (evitar índices innecesarios). También se puede usar técnicas como índices parciales o ajustar el factor de llenado.

---

### Actividad 4: Asignación de roles

| Persona | Tarea | Rol | Justificación |
|---------|-------|-----|---------------|
| Ana | Entrena el modelo de reconocimiento de voz. | **Data Scientist** | Su foco es construir modelos predictivos y algoritmos de ML. |
| Luis | Configura replicación y alta disponibilidad. | **DBA** | Responsable de la administración, respaldos y disponibilidad de la BD. |
| Marta | Construye pipeline ETL para logs. | **Data Engineer** | Diseña y mantiene los flujos de extracción, transformación y carga de datos. |
| Carlos | Crea dashboard de métricas. | **Data Analyst** | Analiza datos y genera visualizaciones para la toma de decisiones. |
| Elena | Define políticas de retención. | **Data Steward** | Asegura la gobernanza, calidad y cumplimiento normativo de los datos. |

---

### Actividad 5: Gobernanza y ética

**a) Protección de perfiles de voz (GDPR)**:
1. **Cifrado en reposo y tránsito**: Almacenar vectores de voz cifrados y usar TLS para todas las comunicaciones.
2. **Minimización de datos**: Solo almacenar lo necesario (ej. vectores, no los audios originales después del enrollment).
3. **Consentimiento explícito**: Solicitar permiso para recoger y procesar la voz, con opción a retirarlo.
4. **Acceso restringido**: Usar RLS y políticas para que solo el usuario y personal autorizado puedan acceder.

**b) Derecho al olvido**:
- El sistema debe eliminar todos los datos asociados al usuario: su perfil de voz, sesiones, mensajes, logs.
- Gracias a las claves foráneas con `ON DELETE CASCADE`, al eliminar un usuario de la tabla `usuarios` se borrarán automáticamente sus registros en `perfiles_voz`, `sesiones`, `mensajes` y `logs_ejecucion` (si se definieron las relaciones correctamente). Esto garantiza la integridad referencial y facilita la operación.

**c) Riesgos éticos**:
- **Exclusión**: Personas con discapacidad del habla, afonía temporal, o condiciones que impidan usar voz quedarían fuera.
- **Seguridad**: Posibilidad de suplantación con grabaciones (deepfakes).
- **Solución inclusiva**: Ofrecer un método alternativo (ej. código de acceso temporal por SMS, o autenticación con token) como opción secundaria, manteniendo la voz como principal pero garantizando accesibilidad.

---

### Actividad 6: Primer contacto con Supabase (opcional)

**Pregunta**: ¿Por qué es importante conocer la estructura de las tablas antes de escribir consultas?
- **Respuesta**: Conocer la estructura (metadatos estructurales) evita errores como referirse a columnas inexistentes, usar tipos de datos incorrectos o unir tablas de manera inapropiada. Además, permite entender las relaciones entre tablas y diseñar consultas eficientes.

---

## 4. Reflexión final

En este laboratorio hemos:
- Aplicado conceptos fundamentales (dato, información, conocimiento, metadatos, hardware, roles, gobernanza) al caso Aletza.
- Reflexionado sobre implicaciones prácticas y éticas.
- Realizado una primera exploración de Supabase (opcional), preparando el terreno para próximos laboratorios donde escribiremos SQL y construiremos la lógica completa del asistente.

**Preparación para la próxima sesión**:
- Revisar el modelo de datos de Aletza (tablas y relaciones).
- Traer dudas sobre normalización y formas normales.

---