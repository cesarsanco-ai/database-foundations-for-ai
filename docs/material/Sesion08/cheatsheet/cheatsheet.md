---
layout: default
---

# Cheatsheet: Pipeline de datos
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-08](../../../sesiones/sesion-08.md)

---

# 🔹 1. OLTP vs OLAP (clave fundamental)

| Característica  | OLTP (Transaccional)   | OLAP (Analítico)     |
| --------------- | ---------------------- | -------------------- |
| Objetivo        | Operaciones diarias    | Análisis y reporting |
| Tipo de queries | Cortas, simples        | Largas, complejas    |
| Escrituras      | Muchas (INSERT/UPDATE) | Pocas (append)       |
| Lecturas        | Puntuales (por ID)     | Agregaciones masivas |
| Datos           | Actual, operativo      | Histórico            |
| Normalización   | Alta (3FN)             | Baja (denormalizado) |
| Latencia        | Milisegundos           | Segundos/minutos     |
| Usuarios        | Apps, usuarios finales | Analistas, BI, ML    |
| Ejemplo         | PostgreSQL, MySQL      | BigQuery, Snowflake  |

👉 Regla rápida:

* **OLTP = escribir rápido**
* **OLAP = leer y analizar rápido**

---

# 🔹 2. Tipos de almacenamiento

## 🏢 Data Warehouse vs Lake vs Lakehouse

| Característica | Data Warehouse  | Data Lake      | Lakehouse |
| -------------- | --------------- | -------------- | --------- |
| Datos          | Estructurados   | Todos          | Todos     |
| Esquema        | Schema-on-write | Schema-on-read | Híbrido   |
| Performance    | Alto            | Medio          | Alto      |
| Costo          | Alto            | Bajo           | Medio     |
| ACID           | Sí              | No (limitado)  | Sí        |

👉 Tendencia actual: **Lakehouse (Databricks, Iceberg, Delta)**

---

# 🔹 3. ETL vs ELT

|              | ETL              | ELT              |
| ------------ | ---------------- | ---------------- |
| Orden        | Transform → Load | Load → Transform |
| Dónde ocurre | Fuera del DW     | Dentro del DW    |
| Latencia     | Alta             | Baja             |
| Flexibilidad | Baja             | Alta             |
| Uso típico   | Legacy/on-prem   | Cloud moderno    |

👉 Regla:

* **ETL = control**
* **ELT = escalabilidad**

---

# 🔹 4. Batch vs Streaming

|             | Batch          | Streaming             |
| ----------- | -------------- | --------------------- |
| Tiempo      | Horas/días     | Tiempo real           |
| Latencia    | Alta           | Baja                  |
| Complejidad | Baja           | Alta                  |
| Ejemplo     | Reporte diario | Fraude en tiempo real |

### Tecnologías

* Batch: Spark, Airflow
* Streaming: Kafka, Flink

👉 Arquitecturas:

* **Lambda**: Batch + Streaming (complejo)
* **Kappa**: Solo streaming (moderno)

---

# 🔹 5. Pipeline de Datos

## 📌 Definiciones clave

* **Pipeline** → flujo completo
* **Job** → proceso (ej: Spark job)
* **Task** → paso individual
* **DAG** → dependencias

---

## 🔁 Pipeline típico

```
Fuente → Ingesta → Staging → Transformación → DW → BI / ML
```

---

# 🔹 6. Capas de un Data Warehouse

| Capa      | Nombre    | Función            |
| --------- | --------- | ------------------ |
| 🟤 Raw    | Bronze    | Datos crudos       |
| ⚪ Staging | Silver    | Limpieza           |
| 🟡 Core   | Gold      | Modelo analítico   |
| 🔵 Mart   | Data Mart | Consumo específico |

👉 Regla:

* Bronze = ingestión
* Silver = limpieza
* Gold = negocio

---

# 🔹 7. Carga incremental (CRÍTICO)

## Técnicas clave

### 1. Watermark

```sql
WHERE fecha > ultima_carga
```

### 2. UPSERT

```sql
INSERT ... ON CONFLICT DO UPDATE
```

### 3. MERGE

```sql
MERGE INTO target USING source ...
```

### 4. CDC (Change Data Capture)

* Captura cambios en tiempo real
* Tools: Debezium + Kafka

👉 Regla:

* **Nunca recargar todo si no es necesario**

---

# 🔹 8. Orquestación

## Herramientas

* Apache Airflow ⭐ (estándar)
* Prefect / Dagster
* Cloud:

  * AWS Step Functions
  * Cloud Composer

---

## Ejemplo mental DAG

```
Extraer → Transformar → Cargar
   ↓           ↓           ↓
  API        Spark       DW
```

---

# 🔹 9. Arquitectura moderna (real)

## Stack típico

* OLTP: PostgreSQL
* Streaming: Kafka
* Procesamiento: Spark / Flink
* Storage: S3 / Data Lake
* DW: Snowflake / BigQuery
* BI: Power BI / Tableau

---

## Flujo real

```
PostgreSQL → Kafka → Spark → S3 → Snowflake → Power BI
```

---

# 🔹 10. Modelado analítico

## Esquema estrella ⭐

* **Fact table** (hechos)
* **Dimension tables**

Ejemplo:

```
ventas (fact)
  - cliente_id
  - producto_id
  - fecha
  - monto

clientes (dim)
productos (dim)
tiempo (dim)
```

---

# 🔹 11. Costos (muy preguntado)

## Factores

* Almacenamiento
* Compute (queries)
* Transferencia

## Regla rápida

* Más queries → más costo
* Más datos → más storage

---

# 🔹 12. Buenas prácticas

✔ Usar particiones por fecha
✔ Usar formatos columnar (Parquet)
✔ Evitar full refresh
✔ Versionar pipelines
✔ Monitorear (logs + métricas)
✔ Diseñar idempotencia

---

# 🔹 13. Errores comunes

❌ No manejar incremental
❌ Mezclar OLTP con OLAP
❌ No tener capa raw
❌ Transformar demasiado temprano
❌ No versionar datos

---

# 🔹 14. Integración con BI

* Conexión: JDBC / ODBC
* Estrategias:

  * Direct Query (tiempo real)
  * Import (más rápido)

👉 Optimización:

* Vistas materializadas
* Tablas agregadas

---

# 🔹 15. Quick mental model (para entrevistas)

Si te preguntan diseño:

👉 Respuesta ideal:

1. Fuente (OLTP)
2. Ingesta (batch/stream)
3. Raw (data lake)
4. Transformación (Spark/SQL)
5. Data Warehouse
6. Data Mart
7. BI / ML

---

# 🔹 16. Resumen ultra rápido

* OLTP = operaciones
* OLAP = análisis
* ETL = transformar antes
* ELT = transformar después
* Batch = por lotes
* Streaming = tiempo real
* DW = estructurado
* Lake = crudo
* Lakehouse = híbrido
* Pipeline = flujo de datos
* Airflow = orquestador

