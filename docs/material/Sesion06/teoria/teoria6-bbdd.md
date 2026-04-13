---
layout: default
---
# Sesion 6: SQL Parte 3 - Window functions y transformaciones avanzadas

### 1. Logro de la sesion

Aplicar SQL avanzado en PostgreSQL para analitica y preparacion de datos usando window functions, transformaciones de texto/fecha, manejo de nulos, CTEs y patrones de feature engineering.

---

### 2. Conexion con el syllabus

Semana 6 corresponde a:

- Window Functions: `OVER`, `RANK`, `ROW_NUMBER`, `LAG`, `LEAD`.
- Feature engineering en SQL.
- Transformaciones avanzadas de fechas, texto y limpieza.

Esta sesion se alinea con `5-transformacion.sql` y `6-analisis.sql`.

---

### 3. De SQL operativo a SQL analitico

En SQL parte 1 y 2 se cubrieron joins, filtros y agregaciones.
En SQL parte 3 el objetivo cambia:

1. Analizar tendencias y comportamiento temporal.
2. Preparar variables para modelos de ML.
3. Generar datasets limpios y consistentes.

---

### 4. Manejo de nulos con criterio

`NULL` significa valor desconocido, no cero ni vacio.

Funciones clave:

- `COALESCE(a,b,...)`.
- `NULLIF(a,b)`.

```sql
SELECT username, COALESCE(pais, 'No especificado') AS pais_normalizado
FROM usuarios;
```

```sql
SELECT modulo, NULLIF(tiempo_ejecucion_ms, 0) AS tiempo_seguro
FROM logs_ejecucion;
```

Buenas practicas:

- Normalizar nulos antes de agregar.
- Evitar mezclar nulo semantico con valores por defecto sin documentar.

---

### 5. Transformaciones de texto

Funciones frecuentes en PostgreSQL:

- `LOWER`, `UPPER`, `INITCAP`.
- `TRIM`, `REPLACE`, `LENGTH`.
- `SUBSTRING`, `SPLIT_PART`.
- `REGEXP_REPLACE`, `REGEXP_MATCHES`.

```sql
SELECT
    username,
    LOWER(email) AS email_normalizado,
    SPLIT_PART(email, '@', 2) AS dominio
FROM usuarios;
```

Uso real:

- limpieza de correo,
- estandarizacion de categorias,
- extraccion de metadatos.

---

### 6. Transformaciones de fecha y hora

Funciones clave:

- `EXTRACT`.
- `DATE_TRUNC`.
- `AGE`.
- `NOW` e `INTERVAL`.

```sql
SELECT
    DATE_TRUNC('month', creado_en) AS mes,
    COUNT(*) AS usuarios_registrados
FROM usuarios
GROUP BY DATE_TRUNC('month', creado_en)
ORDER BY mes;
```

Aplicaciones:

- series temporales,
- cohortes,
- estacionalidad.

---

### 7. Conversion de tipos

Formas en PostgreSQL:

- `CAST(expr AS tipo)`.
- `expr::tipo`.

```sql
SELECT
    tiempo_ejecucion_ms::TEXT AS tiempo_texto,
    enviado_en::DATE AS fecha
FROM logs_ejecucion l
JOIN mensajes m ON l.mensaje_id = m.id;
```

Regla:

Convertir de manera explicita evita ambiguedad y errores de calidad.

---

### 8. Condicionales con `CASE`

`CASE` permite categorizar datos sin salir de SQL.

```sql
SELECT
    modulo,
    tiempo_ejecucion_ms,
    CASE
        WHEN tiempo_ejecucion_ms < 50 THEN 'Rapido'
        WHEN tiempo_ejecucion_ms < 150 THEN 'Normal'
        WHEN tiempo_ejecucion_ms < 300 THEN 'Lento'
        ELSE 'Muy lento'
    END AS categoria
FROM logs_ejecucion;
```

Uso:

- reglas de negocio,
- scoring,
- etiquetas para dashboards.

---

### 9. Window functions: concepto base

Diferencia con `GROUP BY`:

- `GROUP BY` colapsa filas.
- `OVER` conserva detalle y agrega contexto analitico.

Sintaxis general:

```sql
funcion() OVER (PARTITION BY ... ORDER BY ...)
```

---

### 10. Ranking y segmentacion

Funciones:

- `ROW_NUMBER`.
- `RANK`.
- `DENSE_RANK`.
- `NTILE`.

```sql
SELECT
    u.username,
    COUNT(m.id) AS mensajes,
    RANK() OVER (ORDER BY COUNT(m.id) DESC) AS ranking
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username;
```

---

### 11. Analisis secuencial con `LAG` y `LEAD`

Permiten comparar filas consecutivas por entidad.

```sql
SELECT
    remitente_id,
    enviado_en,
    LAG(enviado_en) OVER (PARTITION BY remitente_id ORDER BY enviado_en) AS previo,
    EXTRACT(EPOCH FROM (enviado_en - LAG(enviado_en) OVER (PARTITION BY remitente_id ORDER BY enviado_en))) / 60 AS minutos_diff
FROM mensajes;
```

Casos de uso:

- tiempo entre eventos,
- abandono,
- deteccion de picos.

---

### 12. Acumulados y medias moviles

```sql
SELECT
    DATE(enviado_en) AS fecha,
    COUNT(*) AS mensajes_dia,
    SUM(COUNT(*)) OVER (ORDER BY DATE(enviado_en)) AS acumulado,
    AVG(COUNT(*)) OVER (ORDER BY DATE(enviado_en) ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS media_3
FROM mensajes
GROUP BY DATE(enviado_en)
ORDER BY fecha;
```

Esto habilita analitica de tendencia sin exportar a otra herramienta.

---

### 13. Subconsultas y CTEs

Tipos comunes:

- subconsulta en `SELECT`,
- subconsulta en `WHERE`,
- subconsulta en `FROM`.

CTE (`WITH`) mejora legibilidad y modularidad.

```sql
WITH mensajes_por_usuario AS (
    SELECT remitente_id, COUNT(*) AS total
    FROM mensajes
    GROUP BY remitente_id
)
SELECT u.username, COALESCE(m.total, 0) AS total_mensajes
FROM usuarios u
LEFT JOIN mensajes_por_usuario m ON u.id = m.remitente_id;
```

---

### 14. Feature engineering en SQL

Ejemplos de features para ML:

- frecuencia de mensajes por usuario,
- tiempo promedio entre interacciones,
- modalidad dominante,
- actividad por franja horaria,
- recencia de ultima sesion.

Ventajas de hacerlo en SQL:

- reproducibilidad,
- trazabilidad,
- cercania a los datos fuente.

---

### 15. Calidad de datos para features

Checklist:

1. tipos de datos consistentes,
2. nulos controlados,
3. outliers detectados,
4. llaves unicas verificadas,
5. timestamps con zona horaria clara.

---

### 16. Patrón de consulta analitica robusta

1. filtrar periodo,
2. limpiar campos,
3. derivar features,
4. calcular ventanas,
5. persistir resultado en tabla o vista.

---

### 17. Errores frecuentes en SQL avanzado

- usar ventanas sin `ORDER BY` cuando la secuencia importa,
- mezclar `GROUP BY` y ventanas sin validar grano,
- no documentar reglas de limpieza,
- asumir que `NULL` se comporta como cero,
- crear CTEs excesivos sin revisar rendimiento.

---

### 18. Mini laboratorio

1. Crear ranking de usuarios por actividad.
2. Calcular tiempo entre mensajes por usuario.
3. Construir media movil diaria.
4. Clasificar modulos por performance con `CASE`.
5. Generar tabla de features para modelado.

---

### 19. Checklist de salida

- Domino `OVER` con `PARTITION` y `ORDER`.
- Aplico `LAG/LEAD` en series de eventos.
- Limpio texto y fechas con funciones de PostgreSQL.
- Construyo CTEs claros para analitica.
- Puedo entregar un dataset de features confiable.

---

### 20. Preguntas de autoevaluacion

1. Cuando prefieres window function sobre `GROUP BY`?
2. Que diferencia hay entre `RANK` y `DENSE_RANK`?
3. Como evitar sesgos por nulos en features?
4. Para que sirve `DATE_TRUNC` en cohortes?
5. Que valida primero antes de entrenar un modelo?

---

### 21. Referencias recomendadas

1. PostgreSQL docs: window functions.
2. PostgreSQL docs: date/time functions.
3. PostgreSQL docs: string functions.
4. Material practico de transformacion y analisis.
5. Feature engineering for ML (conceptos aplicados).
