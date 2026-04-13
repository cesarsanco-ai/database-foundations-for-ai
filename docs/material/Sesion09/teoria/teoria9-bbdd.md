---
layout: default
---
# Sesion 9: ETL y pipelines de datos

### 1. Logro de la sesion

Disenar pipelines ETL/ELT confiables para mover datos desde fuentes operativas hacia capas analiticas (Bronze, Silver, Gold), diferenciando batch y streaming, y aplicando controles de calidad, incrementalidad y monitoreo.

---

### 2. Conexion con el syllabus

Semana 9 corresponde a:

- ETL vs ELT.
- Ingesta: APIs, archivos, scraping.
- Batch vs streaming.

Se alinea con `9-etl.sql`.

---

### 3. Que es un pipeline de datos

Un pipeline es una cadena de procesos que:

1. extrae datos,
2. transforma y valida,
3. carga a destino para consumo.

Objetivo real:

entregar datos confiables, oportunos y trazables.

---

### 4. ETL vs ELT

#### 4.1 ETL

Transforma antes de cargar.

Ventajas:

- control fuerte previo,
- destino mas limpio desde inicio.

#### 4.2 ELT

Carga primero, transforma en destino.

Ventajas:

- mayor flexibilidad,
- aprovecha potencia del motor analitico.

---

### 5. Fuentes de datos

Tipos comunes:

- bases relacionales,
- APIs REST,
- archivos CSV/JSON/Parquet,
- logs de aplicaciones,
- scraping controlado.

Recomendacion:

capturar metadata de origen desde la ingesta.

---

### 6. Arquitectura por capas (Medallon)

#### 6.1 Bronze

Datos crudos, historicos, minimo procesamiento.

#### 6.2 Silver

Datos limpiados, tipados y estandarizados.

#### 6.3 Gold

Datos agregados para BI, analitica y ML.

---

### 7. Implementacion en PostgreSQL

Ejemplo de schemas:

```sql
CREATE SCHEMA IF NOT EXISTS bronze;
CREATE SCHEMA IF NOT EXISTS silver;
CREATE SCHEMA IF NOT EXISTS gold;
```

Este patron aparece en los scripts practicos de ETL.

---

### 8. Carga inicial vs incremental

#### 8.1 Carga full

- copia todo,
- simple para arranque,
- costosa en volumen alto.

#### 8.2 Carga incremental

- solo nuevos o cambiados,
- menor costo y latencia,
- requiere clave de cambio (`id`, `updated_at`, CDC).

---

### 9. Marca de agua y control

Tabla de control recomendada:

- proceso,
- ultima ejecucion,
- ultimo id procesado,
- estado,
- registros procesados.

Beneficio:

permite reintentos y auditoria operacional.

---

### 10. Transformaciones tipicas en Silver

1. normalizar texto (`LOWER`, `TRIM`),
2. validar email,
3. extraer dominios,
4. limpiar espacios/ruido,
5. tipar fechas,
6. estandarizar categorias.

---

### 11. Calidad de datos en pipeline

Dimensiones:

- completitud,
- unicidad,
- consistencia,
- validez,
- frescura.

Regla:

si falla control critico, detener carga a Gold.

---

### 12. Batch vs Streaming

#### 12.1 Batch

- corre por ventanas,
- estable y simple,
- mayor latencia.

#### 12.2 Streaming

- casi en tiempo real,
- mejor para eventos continuos,
- mayor complejidad de operacion.

Seleccion segun SLA de negocio.

---

### 13. Orquestacion y scheduling

Un pipeline productivo requiere:

- dependencias entre tareas,
- reintentos,
- alertas,
- registro de ejecucion.

Herramientas comunes:

Airflow, Prefect, Dagster, orquestadores cloud.

---

### 14. Idempotencia en ETL

Un job idempotente se puede re-ejecutar sin duplicar ni corromper datos.

Estrategias:

- `UPSERT` con `ON CONFLICT`,
- deduplicacion por llave natural,
- particionado por ventana temporal.

---

### 15. CDC en pipelines

Change Data Capture detecta cambios en origen para sincronizar destino.

Eventos CDC:

- insert,
- update,
- delete.

Es clave cuando hay integracion con capas vectoriales o microservicios.

---

### 16. Capa Gold y consumo

Objetivo de Gold:

- tablas de hechos y dimensiones,
- resumenes diarios,
- vistas para dashboards y modelos.

No debe contener logica improvisada de ultima hora.

---

### 17. Monitoreo operacional

Metricas minimas:

- duracion del job,
- filas leidas/escritas,
- tasa de error,
- freshness del dataset.

Alertas:

- job fallido,
- retraso de SLA,
- volumen anomalo.

---

### 18. Seguridad y gobierno en ETL

- controlar acceso por capa,
- enmascarar datos sensibles,
- auditar quien transforma que,
- versionar transformaciones.

---

### 19. Errores frecuentes

- pipelines sin tabla de control,
- no manejar retries,
- mezclar datos crudos y curados,
- no validar calidad,
- transformar sin trazabilidad.

---

### 20. Caso aplicado

Plataforma de mensajeria IA:

- Bronze recibe eventos crudos de usuarios y logs.
- Silver limpia textos, fechas y atributos de IA.
- Gold entrega metricas de uso y rendimiento por modulo.

Resultado:

analitica confiable y base de features para ML.

---

### 21. Mini laboratorio

1. Crear schemas Bronze/Silver/Gold.
2. Cargar datos crudos en Bronze.
3. Transformar usuarios y mensajes a Silver.
4. Generar resumen diario en Gold.
5. Registrar ejecucion en tabla de control.

---

### 22. Checklist de salida

- Explico ETL vs ELT segun caso.
- Diseno pipelines incrementales.
- Implemento controles de calidad.
- Distingo batch y streaming por SLA.
- Relaciono ETL con BI y ML.

---

### 23. Preguntas de autoevaluacion

1. Cuando prefieres ETL sobre ELT?
2. Que guarda una tabla de control de cargas?
3. Por que idempotencia evita incidentes?
4. Que control debe bloquear una carga a Gold?
5. Cuando streaming no aporta valor?

---

### 24. Referencias recomendadas

1. Kimball: Data Warehouse Toolkit.
2. Designing Data-Intensive Applications.
3. Docs de orquestadores (Airflow/Prefect/Dagster).
4. Buenas practicas de calidad de datos.
5. Scripts ETL del curso.
