---
layout: default
---
# Sesion 4: SQL Parte 1 - DML, filtros y joins en PostgreSQL

### 1. Logro de la sesion

Construir consultas SQL correctas y legibles en PostgreSQL usando `SELECT`, `INSERT`, `UPDATE`, `DELETE`, filtros con `WHERE` y joins fundamentales para resolver preguntas de negocio sin perder integridad de datos.

---

### 2. Contexto didactico

Esta sesion conecta directamente con:

- `3-insercion.sql`,
- `4-select.sql`,
- `2-modificaciones.sql`.

La teoria debe reflejar la sintaxis usada en la practica.

---

### 3. Anatomia de una consulta `SELECT`

Orden logico de lectura:

1. `FROM`
2. `JOIN`
3. `WHERE`
4. `GROUP BY`
5. `HAVING`
6. `SELECT`
7. `ORDER BY`
8. `LIMIT/OFFSET`

Orden de escritura habitual:

```sql
SELECT columnas
FROM tabla
WHERE condicion
ORDER BY campo
LIMIT n;
```

---

### 4. Proyeccion de columnas

#### 4.1 Evitar `SELECT *` en produccion

Usar columnas explicitas:

```sql
SELECT username, email
FROM usuarios;
```

Beneficios:

- menos transferencia de datos,
- menor acoplamiento ante cambios de esquema,
- mejor lectura del objetivo de consulta.

#### 4.2 Alias de columnas y tablas

```sql
SELECT
    u.username AS nombre_usuario,
    u.email AS correo
FROM usuarios AS u;
```

---

### 5. Filtros con `WHERE`

#### 5.1 Comparadores

- `=`, `!=`, `>`, `<`, `>=`, `<=`.

```sql
SELECT username, pais
FROM usuarios
WHERE pais = 'Perú';
```

#### 5.2 Rango con `BETWEEN`

```sql
SELECT id, enviado_en
FROM mensajes
WHERE enviado_en BETWEEN '2024-01-15' AND '2024-01-16';
```

#### 5.3 Lista con `IN`

```sql
SELECT username, pais
FROM usuarios
WHERE pais IN ('Perú', 'México', 'Argentina');
```

#### 5.4 Patrones con `LIKE` e `ILIKE`

```sql
SELECT username, email
FROM usuarios
WHERE email ILIKE '%@gmail.com';
```

#### 5.5 Nulos

Nunca usar `= NULL`, usar:

- `IS NULL`,
- `IS NOT NULL`.

---

### 6. Logica booleana

Operadores clave:

- `AND`,
- `OR`,
- `NOT`.

Ejemplo compuesto:

```sql
SELECT *
FROM logs_ejecucion
WHERE (modulo = 'clasificacion_nlp' OR modulo = 'api_clima')
  AND exito = TRUE
  AND tiempo_ejecucion_ms > 50;
```

Recomendacion:

Usar parentesis cuando mezcles `AND` y `OR`.

---

### 7. Orden y paginacion

#### 7.1 `ORDER BY`

```sql
SELECT username, pais
FROM usuarios
ORDER BY pais ASC, username ASC;
```

#### 7.2 `NULLS FIRST/LAST`

```sql
SELECT username, pais
FROM usuarios
ORDER BY pais NULLS LAST;
```

#### 7.3 `LIMIT` y `OFFSET`

```sql
SELECT *
FROM usuarios
ORDER BY id
LIMIT 3 OFFSET 2;
```

---

### 8. DML de escritura

#### 8.1 Insercion basica

```sql
INSERT INTO usuarios (username, email, pais)
VALUES ('ana_data', 'ana@example.com', 'Perú');
```

#### 8.2 Insercion desde consulta

```sql
INSERT INTO usuarios_backup (id, username, email, creado_en, pais, ciudad)
SELECT id, username, email, creado_en, pais, ciudad
FROM usuarios
WHERE pais = 'Perú';
```

#### 8.3 Actualizacion

```sql
UPDATE usuarios
SET pais = 'España'
WHERE username = 'carlos_dev';
```

#### 8.4 Borrado

```sql
DELETE FROM sesiones
WHERE expira_en < NOW();
```

---

### 9. Joins fundamentales

#### 9.1 `INNER JOIN`

Devuelve filas con match en ambas tablas.

```sql
SELECT u.username, p.estado
FROM usuarios u
INNER JOIN perfiles_voz p ON u.id = p.usuario_id;
```

#### 9.2 `LEFT JOIN`

Conserva todas las filas de la izquierda.

```sql
SELECT u.username, m.id AS mensaje_id
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id;
```

#### 9.3 `RIGHT JOIN`

Menos usado, equivalente conceptual invirtiendo tablas.

#### 9.4 `FULL JOIN`

Combina no-coincidencias de ambos lados.

---

### 10. Caso guiado de negocio

Pregunta:

"Que usuarios tienen actividad reciente y por que canal?"

Consulta:

```sql
SELECT
    u.username,
    s.canal,
    s.ultima_actividad
FROM usuarios u
JOIN sesiones s ON u.id = s.usuario_id
WHERE s.ultima_actividad > NOW() - INTERVAL '7 days'
ORDER BY s.ultima_actividad DESC;
```

Conceptos usados:

- join,
- filtro temporal,
- orden descendente.

---

### 11. Buenas practicas de estilo SQL

1. Una clausula por linea.
2. Alias cortos y consistentes.
3. Palabras clave en mayusculas.
4. Condiciones complejas con indentacion clara.
5. Comentarios solo cuando agregan contexto.

---

### 12. Errores frecuentes

- olvidar `WHERE` en `UPDATE`/`DELETE`,
- usar `SELECT *` en reportes productivos,
- comparar con NULL usando `=`,
- paginar sin `ORDER BY`,
- hacer joins sin condicion de union.

---

### 13. Integridad y seguridad minima

Antes de ejecutar escritura:

1. probar con `SELECT` previo,
2. envolver cambios masivos en transaccion,
3. confirmar cantidad de filas afectadas.

Ejemplo:

```sql
BEGIN;

UPDATE perfiles_voz
SET muestras_tomadas = muestras_tomadas + 1
WHERE estado = 'incompleto';

-- revisar resultado
SELECT COUNT(*) FROM perfiles_voz WHERE estado = 'incompleto';

COMMIT;
```

---

### 14. Mini laboratorio alineado a scripts

#### 14.1 Objetivo

Replicar un flujo completo:

- consultar,
- filtrar,
- unir,
- actualizar.

#### 14.2 Actividades

1. Listar usuarios por pais (`WHERE`, `ORDER BY`).
2. Obtener sesiones por canal (`IN`).
3. Unir usuarios y perfiles (`INNER JOIN`).
4. Actualizar estado de perfil (`UPDATE`).
5. Verificar cambios con consulta final.

---

### 15. Checklist de dominio

- Identifico cuando usar `INNER` vs `LEFT`.
- Escribo filtros correctos con NULL.
- Domino paginacion con `LIMIT/OFFSET`.
- Ejecuto inserciones desde `SELECT`.
- Actualizo datos con condiciones seguras.

---

### 16. Preguntas de autoevaluacion

1. Que diferencia semantica hay entre `INNER JOIN` y `LEFT JOIN`?
2. Cuando `ILIKE` es preferible a `LIKE`?
3. Que riesgo hay en paginar sin orden?
4. Por que conviene escribir columnas explicitas?
5. Como validas un `UPDATE` masivo antes de confirmar?

---

### 17. Referencias recomendadas

1. PostgreSQL Documentation: `SELECT`.
2. PostgreSQL Documentation: Data Manipulation.
3. Mode SQL Tutorial: Joins and Filters.
4. Celko, J. SQL for Smarties.
5. Material practico: scripts de sesion SQL.

