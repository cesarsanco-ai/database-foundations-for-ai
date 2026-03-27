# Laboratorio 3: Normalización e Integridad en Aletza

## Objetivo

Aplicar los conceptos de normalización (1FN, 2FN, 3FN, BCNF) y restricciones de integridad (dominio, entidad, referencial) al caso de estudio **Aletza**, sin escribir código SQL aún. El estudiante analizará el modelo de datos existente, identificará posibles anomalías, propondrá mejoras y reflexionará sobre las decisiones de diseño que garantizan la consistencia de los datos en un sistema real.

---

## 1. Introducción Teórica Breve

La **normalización** es el proceso de organizar los atributos en tablas para eliminar redundancias y prevenir anomalías de inserción, actualización y eliminación. Se basa en **dependencias funcionales** y aplica reglas conocidas como formas normales.

- **1FN**: Atributos atómicos, sin grupos repetitivos.
- **2FN**: En tablas con clave compuesta, todos los atributos no clave dependen completamente de la clave.
- **3FN**: Eliminar dependencias transitivas (atributos no clave que dependen de otros atributos no clave).
- **BCNF**: Toda dependencia funcional no trivial tiene como determinante una superclave.

La **integridad de datos** se garantiza mediante restricciones:

- **Dominio**: `NOT NULL`, `CHECK`, tipos de datos.
- **Entidad**: `PRIMARY KEY`, `UNIQUE`.
- **Referencial**: `FOREIGN KEY` con acciones (`CASCADE`, `SET NULL`, etc.).

---

## 2. Actividades

### Actividad 1: Identificar violaciones de normalización en el modelo actual de Aletza

El modelo inicial de Aletza (propuesto en el caso de estudio) incluye las siguientes tablas:

#### Tabla: usuarios
| Campo        |
|-------------|
| id          |
| username    |
| email       |
| creado_en   |

#### Tabla: perfiles_voz
| Campo           |
|----------------|
| id              |
| usuario_id      |
| palabra_magica  |
| vector_voz      |
| muestras_tomadas|
| estado          |
| creado_en       |

#### Tabla: sesiones
| Campo           |
|----------------|
| id              |
| usuario_id      |
| token           |
| canal           |
| iniciada_en     |
| ultima_actividad|
| expira_en       |

#### Tabla: mensajes
| Campo        |
|-------------|
| id          |
| sesion_id   |
| remitente_id|
| contenido   |
| modalidad   |
| metadata_ia |
| enviado_en  |

#### Tabla: logs_ejecucion
| Campo           |
|----------------|
| id              |
| mensaje_id      |
| modulo          |
| tiempo_ejecucion_ms |
| exito           |
| error           |
| ejecutado_en    |

Analiza cada tabla y responde:

1. **¿Alguna tabla viola la 1FN?** Justifica.
2. **¿Hay tablas con clave compuesta? Si es así, ¿puede haber dependencias parciales que violen 2FN?** Examina especialmente `logs_ejecucion`.
3. **¿Existen dependencias transitivas en alguna tabla?** Por ejemplo, ¿algún atributo no clave determina otro atributo no clave?
4. **¿La tabla `perfiles_voz` está en 3FN?** Considera que `palabra_magica` siempre es "aletza". ¿Podría almacenarse en otro lugar?

### Solucionario Actividad 1: Identificar violaciones de normalización

1. **1FN**: Todas las tablas cumplen 1FN porque cada columna almacena valores atómicos. No hay grupos repetidos.  
   - `perfiles_voz.vector_voz` es JSONB, pero es un valor único (un documento JSON), no una lista repetitiva de valores simples. Aunque JSONB puede contener arrays, se considera un valor compuesto pero atómico desde la perspectiva del modelo relacional (es un tipo de datos). En la práctica, es aceptable porque no se almacenan múltiples valores en una columna con separadores. Si dentro del JSON hubiera un array de muestras individuales, eso sí violaría 1FN. En nuestro diseño actual, `vector_voz` almacena un único embedding final, no las 10 muestras.

2. **Clave compuesta**: Ninguna tabla tiene clave primaria compuesta.  
   - `logs_ejecucion` tiene `id` como PK simple, por lo que no aplica 2FN. Si la clave fuera `(mensaje_id, modulo)`, entonces sí habría que verificar dependencias parciales, pero no es el caso.

3. **Dependencias transitivas**:  
   - En `sesiones`, `usuario_id` determina al usuario, pero no hay otros atributos no clave que dependan de otro no clave.  
   - En `mensajes`, `sesion_id` determina al usuario implícitamente (a través de la sesión), pero no hay atributos no clave que dependan de otro no clave dentro de la misma tabla.  
   - En `logs_ejecucion`, `mensaje_id` determina el mensaje, pero no hay atributos adicionales dependientes.  
   - En `perfiles_voz`, `palabra_magica` es siempre "aletza". Esto podría considerarse redundante, pero no es una dependencia transitiva porque no hay otra columna que la determine.

4. **Perfiles_voz en 3FN**: La tabla está en 3FN porque todos los atributos no clave (`palabra_magica`, `vector_voz`, `muestras_tomadas`, `estado`, `creado_en`) dependen directamente de la clave `id` (o de `usuario_id` si consideramos que `usuario_id` también es clave candidata). No hay dependencias transitivas. Sin embargo, la columna `palabra_magica` es constante para todos los registros, lo que podría ser una redundancia que se puede eliminar moviéndola a una tabla de configuración, pero no es una violación de forma normal.

---

### Actividad 2: Normalización de `perfiles_voz`

Actualmente `perfiles_voz` almacena las muestras de voz como un único campo `vector_voz` (JSONB) que contiene el embedding biométrico después de las 10 muestras. Sin embargo, el proceso de enrollment requiere almacenar **cada muestra individual** para poder entrenar el perfil.

**Pregunta:**  
¿Cómo modificarías el modelo para registrar las 10 muestras individuales? Propón un esquema normalizado que respete 1FN y 2FN. ¿Qué nuevas tablas crearías y cuáles serían sus claves?


### Solucionario actividad 2: Normalización de `perfiles_voz`

Para registrar cada muestra individual, se necesita una tabla separada:

- `muestra_voz (id, perfil_id, muestra_numero, vector_muestra, creado_en)`
- `perfiles_voz` se simplifica: `(id, usuario_id, estado, creado_en)`  
- `muestras_tomadas` se puede derivar contando las muestras, pero puede mantenerse como atributo derivado (o no).

La nueva tabla `muestra_voz` tiene clave primaria `id` (o `(perfil_id, muestra_numero)`). La clave foránea `perfil_id` referencia a `perfiles_voz`.  
Esto cumple 1FN (cada muestra es una fila) y 2FN (la clave es simple o compuesta, pero no hay dependencias parciales).

---

### Actividad 3: Dependencias funcionales en `sesiones`

Supón que se añade un campo `nombre_canal` a `sesiones` para almacenar el nombre descriptivo del canal (ej. "Telegram Bot", "Web App", "iOS App"). Observa que `canal` es un código abreviado ('telegram', 'web', 'app') y `nombre_canal` sería su descripción.

1. **Identifica la dependencia funcional que surge.**
2. **¿Esta dependencia viola alguna forma normal? ¿Cuál y por qué?**
3. **Propón una solución normalizada.**


### Solucionario actividad 3: Dependencias funcionales en `sesiones`

1. **Dependencia funcional**: `canal → nombre_canal` (el código abreviado determina el nombre descriptivo).
2. **Violación**: Si `nombre_canal` se añade a `sesiones`, se introduce una dependencia transitiva si `canal` no es clave. `canal` no es clave primaria ni candidata (puede haber muchas sesiones con el mismo canal). Entonces, `nombre_canal` depende de `canal`, que no es superclave. Esto viola 3FN (y también BCNF).
3. **Solución**: Crear una tabla `canal` con `(codigo, nombre)` y en `sesiones` mantener solo `canal` como FK. Esto normaliza.

---

### Actividad 4: Restricciones de integridad (dominio y entidad)

Define restricciones de dominio apropiadas para los siguientes campos en el modelo de Aletza. No escribas código, pero describe qué tipo de restricción usarías y los valores permitidos.

- `modalidad` en `mensajes`
- `estado` en `perfiles_voz`
- `muestras_tomadas` en `perfiles_voz` (rango)
- `canal` en `sesiones`
- `exito` en `logs_ejecucion`
- `token` en `sesiones` (unicidad)

Además, ¿qué restricción de entidad (clave primaria) asegura que no haya dos usuarios con el mismo email? ¿Y con el mismo username?

### Solucionario actividad 4: Restricciones de integridad

- `modalidad`: `CHECK (modalidad IN ('texto', 'audio', 'imagen', 'video'))`
- `estado`: `CHECK (estado IN ('incompleto', 'completo'))`
- `muestras_tomadas`: `CHECK (muestras_tomadas BETWEEN 0 AND 10)`
- `canal`: `CHECK (canal IN ('telegram', 'web', 'app'))`
- `exito`: `BOOLEAN` (ya restringido por tipo)
- `token`: `UNIQUE` (además de `NOT NULL`)
- **Entidad**: `PRIMARY KEY` en cada tabla asegura unicidad. Para email y username: `UNIQUE` en `usuarios`.
---

### Actividad 5: Integridad referencial en el proceso de enrollment

El proceso de enrollment requiere que un usuario registre 10 muestras de voz. El modelo actual almacena el resultado final en `perfiles_voz` y el campo `muestras_tomadas` cuenta cuántas se han recibido.

**Reflexiona:**

- Si se eliminara un usuario, ¿qué debería ocurrir con su perfil de voz? ¿Y con sus sesiones y mensajes? ¿Qué acción referencial (`CASCADE`, `SET NULL`, `RESTRICT`) sería adecuada en cada FK?
- Durante el enrollment, ¿cómo se garantiza que no se excedan las 10 muestras? ¿Es una restricción de dominio o una regla de negocio que requiere lógica adicional?
- Propón una regla de negocio (conceptualmente) para evitar que un usuario complete el enrollment si no ha enviado exactamente 10 muestras.

### Solucionario actividad 5: Integridad referencial en el enrollment

- **Eliminación de usuario**: Debería eliminarse en cascada sus perfiles de voz, sesiones, mensajes y logs. Las FK deberían tener `ON DELETE CASCADE`.
- **Control de 10 muestras**: Es una regla de negocio que puede implementarse con un trigger o en la lógica de la aplicación. No es una restricción declarativa simple porque implica contar registros de otra tabla. Se puede usar un `CHECK` con una subconsulta en algunos SGBD, pero no es común; mejor se implementa con un trigger que impida la inserción de la muestra 11.
- **Evitar completar sin 10 muestras**: Se puede definir que el `estado` solo pueda cambiar a 'completo' cuando `muestras_tomadas = 10`. Esto podría ser un trigger o una restricción `CHECK` que valide la condición en la actualización.
---

### Actividad 6: Anomalías de actualización en un modelo desnormalizado

Imagina que en lugar de tener tablas separadas para `usuarios` y `perfiles_voz`, se hubiera optado por guardar toda la información en una sola tabla `usuarios_con_perfil` con campos: `id`, `username`, `email`, `vector_voz`, `muestras_tomadas`, `estado`, `palabra_magica`.

1. **¿Qué anomalías de actualización podrían ocurrir si un usuario cambia su email?** (Considera que el email aparece una sola vez por usuario, pero el problema es otro: en este diseño, el email se almacena una sola vez, pero si hubiera múltiples filas por usuario, sería peor. En realidad, la anomalía sería la redundancia de otros datos si se repite el perfil, pero aquí hay una fila por usuario, no hay múltiples filas. Sin embargo, el diseño sigue siendo redundante porque el perfil de voz es un solo registro por usuario, pero podría tener sentido. La pregunta busca identificar que la desnormalización no siempre es mala, pero debemos pensar en si hubiera repetición).

   Mejor replantea: supón que en lugar de una tabla `perfiles_voz` separada, los vectores de voz se almacenan en la misma tabla `usuarios`. ¿Qué problemas de integridad podrían surgir si un usuario tuviera varios perfiles (por ejemplo, por actualización de voz) y no se separara?

   Reformula para reflejar una desnormalización típica: en lugar de `perfiles_voz` con FK, se guarda `vector_voz` en `usuarios`. Pero el proceso de enrollment requiere almacenar muestras individuales. Si se guardaran todas las muestras en una columna repetitiva, violaría 1FN.

   La actividad busca que el estudiante identifique las ventajas de tener tablas separadas para manejar el ciclo de vida del perfil.

**Enunciado concreto:**  
Supón que, por simplicidad, se decide eliminar la tabla `perfiles_voz` y añadir los campos `vector_voz`, `muestras_tomadas`, `estado` y `palabra_magica` directamente en `usuarios`.  
- ¿Qué ocurre si un usuario necesita actualizar su perfil de voz (por ejemplo, porque su voz cambia con el tiempo)?  
- ¿Cómo afecta esto al historial de autenticaciones?  
- ¿Qué anomalías de actualización o eliminación podrían presentarse si más adelante se quisiera guardar un histórico de perfiles?


### Solucionario actividad 6: Anomalías de actualización en un modelo desnormalizado

Si se guarda todo en una sola tabla `usuarios` con los campos del perfil de voz:

- **Actualización de perfil**: Si el usuario necesita reentrenar su voz (por ejemplo, después de una enfermedad), se perdería el perfil anterior. No habría historial.
- **Historial de autenticaciones**: No se podría saber con qué perfil se autenticó en el pasado si el perfil cambia.
- **Anomalías de actualización**: Si más adelante se quisiera mantener un histórico de perfiles (por ejemplo, para auditoría), sería necesario duplicar los datos del usuario en varias filas, lo que violaría 1FN o causaría redundancia. La solución es tener tablas separadas que permitan múltiples perfiles a lo largo del tiempo.

---

### Actividad 7: Dependencias multivaluadas y 4NF

En el sistema Aletza, cada mensaje puede tener varios módulos de IA aplicados, y cada módulo genera un registro en `logs_ejecucion`. Actualmente, `logs_ejecucion` tiene una clave primaria `id` simple, y cada fila representa un módulo.

Supón que un mensaje también pudiera estar asociado a varios destinatarios (por ejemplo, un mensaje enviado a un grupo). Si se añadiera una tabla `mensaje_destinatario` con `(id_mensaje, id_usuario)`, ¿se violaría alguna forma normal? Explica.

### Solucionario actividad 7: Dependencias multivaluadas y 4NF

Si se añade `mensaje_destinatario` con `(id_mensaje, id_usuario)`, se introduce una dependencia multivaluada: un mensaje puede tener varios destinatarios independientemente de otros atributos. Si el mensaje también tuviera, por ejemplo, varias etiquetas (categorías), y ambas fueran independientes, se violaría 4NF. En nuestro caso, con solo una relación multivalorada (destinatarios), no hay problema, pero si más adelante se añadiera otra tabla similar (ej. `mensaje_etiqueta`), habría que considerar si las dos son independientes. Si lo son, la combinación en una sola tabla generaría redundancia y violaría 4NF.




