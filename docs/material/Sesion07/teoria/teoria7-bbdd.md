---
layout: default
---
# Sesion 7: Optimizacion de consultas en PostgreSQL

### 1. Logro de la sesion

Analizar cuellos de botella de consultas SQL usando `EXPLAIN ANALYZE`, comprender como el optimizador elige planes de ejecucion y aplicar estrategias de indexacion (B-Tree, GIN, funcional y parcial) con criterio costo-beneficio.

---

### 2. Vinculo con practica

Esta sesion se alinea con `7-optimizacion.sql`, especialmente:

- lectura de planes,
- creacion de indices,
- medicion antes y despues,
- uso de `pg_stat_statements`.

---

### 3. Principio base de performance

El tiempo total de consulta depende de:

1. costo de leer datos,
2. costo de unir/agregar,
3. cantidad de filas intermedias,
4. calidad de estadisticas del planner.

No existe optimizacion "universal", siempre depende del patron de carga.

---

### 4. Complejidad en bases de datos

#### 4.1 Lectura secuencial

- buena para tablas pequenas,
- buena para escaneos completos,
- mala para busquedas selectivas en tablas grandes.

#### 4.2 Lectura indexada

- excelente para filtros altamente selectivos,
- requiere mantenimiento en escrituras,
- puede degradarse con baja selectividad.

#### 4.3 Big O orientativo

- busqueda secuencial cercana a O(n),
- indexacion B-Tree aproximada O(log n) en acceso.

---

### 5. `EXPLAIN` y `EXPLAIN ANALYZE`

#### 5.1 Diferencia

- `EXPLAIN`: plan estimado.
- `EXPLAIN ANALYZE`: plan ejecutado con tiempos reales.

#### 5.2 Uso recomendado

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT u.username, COUNT(m.id)
FROM usuarios u
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username;
```

#### 5.3 Que leer primero

1. tipo de scan (`Seq Scan`, `Index Scan`),
2. filas estimadas vs reales,
3. nodo mas costoso,
4. tiempo total y buffers.

---

### 6. Operadores de plan mas comunes

#### 6.1 `Seq Scan`

Escaneo completo de tabla.

#### 6.2 `Index Scan`

Lectura por indice y acceso a heap.

#### 6.3 `Bitmap Heap Scan`

Intermedio util en filtros con varias paginas.

#### 6.4 `Nested Loop`

Muy eficiente con conjuntos pequenos e indices buenos.

#### 6.5 `Hash Join`

Frecuente para joins medianos y grandes.

#### 6.6 `Merge Join`

Eficiente con entradas ordenadas.

---

### 7. Estadisticas y planner

PostgreSQL decide con estadisticas, no con intuicion del usuario.

Comandos clave:

```sql
ANALYZE;
VACUUM (ANALYZE);
```

Si las estadisticas estan desactualizadas:

- el planner puede elegir mal,
- aparecen planes inestables,
- sube latencia sin cambios aparentes de SQL.

---

### 8. Indices en PostgreSQL

#### 8.1 B-Tree

Default para igualdad, rangos y orden.

```sql
CREATE INDEX idx_usuarios_pais ON usuarios(pais);
```

#### 8.2 Indice compuesto

```sql
CREATE INDEX idx_mensajes_remitente_modalidad
ON mensajes(remitente_id, modalidad);
```

Orden de columnas importa.

#### 8.3 Indice parcial

```sql
CREATE INDEX idx_perfiles_completos
ON perfiles_voz(usuario_id)
WHERE estado = 'completo';
```

Reduce tamano cuando solo consultas un subconjunto.

#### 8.4 Indice funcional

```sql
CREATE INDEX idx_usuarios_email_lower
ON usuarios(LOWER(email));
```

Necesario cuando el filtro aplica funcion.

#### 8.5 GIN para JSONB

```sql
CREATE INDEX idx_mensajes_metadata_gin
ON mensajes USING gin(metadata_ia);
```

Util para busqueda en documentos JSON.

---

### 9. Medir antes y despues

Regla:

1. medir baseline,
2. aplicar cambio,
3. medir mismo query y datos,
4. comparar no solo tiempo, tambien plan.

Ejemplo:

```sql
EXPLAIN ANALYZE
SELECT username, email
FROM usuarios
WHERE pais = 'Perú';
```

Esperado tras indice:

- menor costo total,
- menos bloques leidos,
- plan indexado cuando corresponde.

---

### 10. Trade-offs de indexar

Beneficios:

- menor latencia de lectura selectiva,
- joins mas rapidos.

Costos:

- inserciones y updates mas lentos,
- mayor uso de disco,
- mayor trabajo de mantenimiento.

Principio:

No indexar por moda, indexar por evidencia.

---

### 11. Optimizacion de consultas SQL

#### 11.1 Seleccionar solo columnas necesarias

Evita `SELECT *` en queries calientes.

#### 11.2 Reducir filas temprano

Filtrar antes de agregar/joinear cuando sea posible.

#### 11.3 Evitar funciones no indexables sin estrategia

`LOWER(col)` requiere indice funcional.

#### 11.4 Controlar CTE y subconsultas

En versiones modernas, CTE suele inlinearse, pero revisar plan siempre.

---

### 12. `pg_stat_statements`

Permite identificar consultas costosas por:

- tiempo total,
- tiempo promedio,
- numero de llamadas.

Consulta base:

```sql
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

Uso pedagogico:

Construir backlog real de optimizacion.

---

### 13. Caso aplicado con JSONB

Consulta:

```sql
SELECT contenido, metadata_ia->>'intencion' AS intencion
FROM mensajes
WHERE metadata_ia->>'sentimiento' = 'positivo';
```

Si escala volumen:

- crear indice funcional sobre expresion,
- o GIN sobre columna JSONB segun patron.

---

### 14. Checklist para diagnostico

1. El problema es SQL, datos o infraestructura?
2. Tengo plan real con `ANALYZE`?
3. Las estimaciones difieren mucho de filas reales?
4. Existen indices alineados al filtro/orden/join?
5. El cambio mejora lectura sin romper escritura?

---

### 15. Errores frecuentes

- crear demasiados indices redundantes,
- medir con cache caliente sin control,
- optimizar sin carga representativa,
- asumir que `CTE` siempre mejora,
- ignorar mantenimiento (`VACUUM/ANALYZE`).

---

### 16. Mini laboratorio

#### 16.1 Objetivo

Optimizar cinco consultas del entorno de mensajeria IA.

#### 16.2 Secuencia

1. Obtener plan inicial.
2. Clasificar nodo dominante.
3. Proponer indice.
4. Re-ejecutar plan.
5. Documentar ganancia y costo.

#### 16.3 Evidencia minima

- captura de plan antes,
- captura de plan despues,
- comentario tecnico de la decision.

---

### 17. Rubrica de evaluacion

- Diagnostico correcto del cuello: 30%.
- Propuesta de indice adecuada: 30%.
- Validacion con metrica: 25%.
- Argumento de trade-off: 15%.

---

### 18. Preguntas de autoevaluacion

1. Cuando prefieres `Seq Scan` aunque exista indice?
2. Que indica una gran diferencia entre filas estimadas y reales?
3. Que tipo de indice usar para `LOWER(email)`?
4. Que costo principal trae indexar en exceso?
5. Para que sirve `pg_stat_statements` en un equipo de datos?

---

### 19. Referencias recomendadas

1. PostgreSQL Docs: `EXPLAIN`.
2. PostgreSQL Docs: Indexes.
3. PostgreSQL Docs: Performance Tips.
4. Use the Index, Luke! (Markus Winand).
5. Material practico de optimizacion del curso.

