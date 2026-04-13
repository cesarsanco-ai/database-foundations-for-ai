---
layout: default
---
# Sesion 3: Normalizacion, integridad y desnormalizacion para analitica

### 1. Logro de la sesion

Disenar esquemas relacionales en PostgreSQL que cumplan 1FN, 2FN y 3FN, aplicar constraints de integridad con criterio de negocio y decidir cuando desnormalizar de forma controlada para cargas analiticas e IA.

---

### 2. Mapa de la sesion

- Parte A: problema de redundancia y anomalias.
- Parte B: 1FN, 2FN, 3FN con ejemplos.
- Parte C: integridad y constraints en PostgreSQL.
- Parte D: desnormalizacion orientada a performance.
- Parte E: puente con ETL y modelado analitico.

---

### 3. Por que normalizar

#### 3.1 Problema inicial

Cuando una sola tabla mezcla usuarios, pedidos, pagos y soporte:

- se repiten datos de cliente en muchas filas,
- aumenta el riesgo de inconsistencia,
- se dificulta actualizar sin romper historial.

#### 3.2 Anomalias clasicas

1. Anomalia de insercion: no puedo crear un cliente sin pedido.
2. Anomalia de actualizacion: cambio de email en una fila, pero no en todas.
3. Anomalia de borrado: elimino ultimo pedido y pierdo informacion del cliente.

#### 3.3 Objetivo real

Normalizar no es "hacer muchas tablas", es:

- separar entidades,
- definir dependencias correctas,
- proteger consistencia de datos.

---

### 4. Primera forma normal (1FN)

#### 4.1 Regla de 1FN

Cada columna debe tener valores atomicos, sin listas ni grupos repetidos.

#### 4.2 Anti patron

Tabla `clientes` con columna `telefonos = '999111222,988777666'`.

#### 4.3 Diseno correcto

- Tabla `clientes`.
- Tabla `cliente_telefonos`.
- Relacion por `cliente_id`.

#### 4.4 Ejemplo PostgreSQL

```sql
CREATE TABLE clientes (
    cliente_id BIGSERIAL PRIMARY KEY,
    nombre TEXT NOT NULL,
    email TEXT UNIQUE
);

CREATE TABLE cliente_telefonos (
    telefono_id BIGSERIAL PRIMARY KEY,
    cliente_id BIGINT NOT NULL REFERENCES clientes(cliente_id),
    telefono TEXT NOT NULL
);
```

#### 4.5 Criterio docente

Si un campo puede contener "varios elementos", probablemente requiere tabla hija.

---

### 5. Segunda forma normal (2FN)

#### 5.1 Regla de 2FN

Debe estar en 1FN y no tener dependencias parciales de una clave compuesta.

#### 5.2 Caso tipico

Tabla `detalle_pedido(pedido_id, producto_id, producto_nombre, cantidad)`.

`producto_nombre` depende de `producto_id`, no de toda la clave compuesta.

#### 5.3 Solucion

- mover atributos de producto a tabla `productos`,
- dejar en `detalle_pedido` solo datos del hecho de compra.

#### 5.4 Resultado esperado

- menor redundancia,
- menor costo de mantenimiento,
- mejor calidad para analitica.

---

### 6. Tercera forma normal (3FN)

#### 6.1 Regla de 3FN

Debe estar en 2FN y no tener dependencias transitivas de la clave.

#### 6.2 Ejemplo

`empleados(empleado_id, area_id, area_nombre, jefe_area)`.

`area_nombre` y `jefe_area` dependen de `area_id`, no de `empleado_id`.

#### 6.3 Solucion

- tabla `areas`,
- tabla `empleados` con FK a `areas`.

#### 6.4 Beneficio

Un cambio de jefe de area se realiza una sola vez.

---

### 7. Integridad de datos en PostgreSQL

#### 7.1 Integridad de entidad

Toda tabla operacional debe tener PK estable.

```sql
CREATE TABLE usuarios (
    id BIGSERIAL PRIMARY KEY,
    username TEXT NOT NULL UNIQUE
);
```

#### 7.2 Integridad referencial

Las FK conectan hechos con dimensiones o maestros.

```sql
CREATE TABLE sesiones (
    id BIGSERIAL PRIMARY KEY,
    usuario_id BIGINT NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE
);
```

#### 7.3 Integridad de dominio

Reglas de rango, formato o conjunto valido.

```sql
ALTER TABLE sesiones
ADD CONSTRAINT canal_valido
CHECK (canal IN ('web', 'telegram', 'app'));
```

#### 7.4 Integridad semantica

Reglas mas ricas que combinan negocio y tiempo:

- `expira_en > iniciada_en`,
- `muestras_tomadas BETWEEN 0 AND 10`,
- `email` con patron valido.

---

### 8. Constraints que debes dominar

#### 8.1 `PRIMARY KEY`

- identifica de forma unica,
- no admite NULL,
- crea indice implicito.

#### 8.2 `FOREIGN KEY`

- asegura correspondencia entre tablas,
- evita huerfanos,
- permite cascadas controladas.

#### 8.3 `UNIQUE`

- evita duplicados en campos candidatos,
- util para `email`, `dni`, `token`.

#### 8.4 `NOT NULL`

- evita ambiguedad semantica,
- obliga completitud minima.

#### 8.5 `CHECK`

- valida reglas locales de columna o fila,
- reduce errores antes de llegar al ETL.

---

### 9. Normalizacion y consultas SQL

#### 9.1 Impacto positivo

- joins mas semanticos,
- agregaciones mas confiables,
- menos correcciones ad hoc.

#### 9.2 Impacto operativo

Normalizar incrementa joins en lectura, pero mejora coherencia general.

#### 9.3 Ejemplo conectado a scripts

Desde los scripts de practica (`4-select.sql` y `6-analisis.sql`):

```sql
SELECT u.username, COUNT(m.id) AS total_mensajes
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.username;
```

Este patron depende de un modelo bien normalizado.

---

### 10. Cuando desnormalizar

#### 10.1 No es un pecado tecnico

Desnormalizar puede ser correcto en lectura analitica intensiva.

#### 10.2 Casos validos

- dashboards con SLA estricto,
- features para ML con cientos de joins,
- reportes periodicos de alto volumen.

#### 10.3 Formas comunes

- tabla resumen diaria,
- materialized views,
- tablas denormalizadas de consumo.

#### 10.4 Regla de oro

Primero modelo fuente consistente, despues capa de rendimiento.

---

### 11. Relacion con ETL y arquitectura medallon

#### 11.1 Bronze

Datos crudos, poco o nulo control de calidad.

#### 11.2 Silver

Normalizacion funcional y limpieza.

#### 11.3 Gold

Desnormalizacion orientada a consulta de negocio.

#### 11.4 Puente curricular

Esta sesion prepara las semanas de ETL y arquitecturas hibridas.

---

### 12. Errores frecuentes del estudiante

1. Confundir PK tecnica con clave de negocio.
2. Duplicar atributos de catalogo en tablas transaccionales.
3. No definir `ON DELETE` en FK.
4. Usar JSONB para todo sin necesidad.
5. Saltar reglas de dominio por "rapidez".

---

### 13. Mini laboratorio guiado

#### 13.1 Objetivo

Refactorizar un esquema con redundancia.

#### 13.2 Pasos

1. Crear tabla no normalizada de ejemplo.
2. Detectar anomalias.
3. Proponer 1FN, 2FN, 3FN.
4. Implementar PK, FK, CHECK.
5. Validar con consultas de agregacion.

#### 13.3 Criterio de evaluacion

- consistencia,
- claridad del modelo,
- calidad de constraints.

---

### 14. Checklist de salida

- Puedo explicar diferencia entre 1FN, 2FN, 3FN.
- Puedo detectar dependencia parcial y transitiva.
- Puedo escribir constraints basicos en PostgreSQL.
- Puedo justificar una desnormalizacion con evidencia.
- Puedo relacionar modelo transaccional con capa analitica.

---

### 15. Preguntas de autoevaluacion

1. Que anomalia aparece si borro la ultima venta de un cliente?
2. En que caso `UNIQUE` no reemplaza a una PK?
3. Por que 3FN reduce deuda de datos?
4. Que riesgo tiene desnormalizar en origen OLTP?
5. Donde ubicas una tabla resumen: fuente o consumo?

---

### 16. Referencias recomendadas

1. Codd, E. F. A Relational Model of Data.
2. Date, C. J. Database Design and Relational Theory.
3. PostgreSQL Docs: Constraints.
4. Elmasri y Navathe: Fundamentos de Bases de Datos.
5. Kimball y Ross: modelado dimensional.

