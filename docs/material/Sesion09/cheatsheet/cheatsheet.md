---
layout: default
---

# Cheatsheet: Big Data
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-09](../../../sesiones/sesion-09.md)

---

## 🔥 1. Las 5 V del Big Data

| V         | Qué significa           | Ejemplo              |
| --------- | ----------------------- | -------------------- |
| Volumen   | Muchísimos datos        | TB, PB               |
| Velocidad | Datos en tiempo real    | IoT, pagos           |
| Variedad  | Diferentes formatos     | JSON, video          |
| Veracidad | Calidad / confiabilidad | datos inconsistentes |
| Valor     | Utilidad para negocio   | insights             |

👉 **Regla clave:**
Si **una sola máquina NO basta → es Big Data**

---

## 🧱 2. Formatos de Datos (CRÍTICO 🚨)

### 📊 Comparativa principal

| Formato | Tipo     | Compresión | Velocidad lectura | Uso ideal             | Evolución schema |
| ------- | -------- | ---------- | ----------------- | --------------------- | ---------------- |
| Parquet | Columnar | 🔥 Alta    | 🔥🔥 Muy alta     | Analytics / Data Lake | Sí               |
| ORC     | Columnar | 🔥 Alta    | 🔥🔥 Muy alta     | Hive/Hadoop           | Sí               |
| Avro    | Fila     | Media      | Media             | Streaming / Kafka     | Sí               |
| JSON    | Texto    | Baja       | Baja              | APIs                  | No               |
| CSV     | Texto    | Baja       | Baja              | Simple                | No               |

---

### ⚡ Regla rápida de uso

* 📊 **Parquet → análisis (BI, Spark)**
* 📡 **Avro → streaming (Kafka)**
* 📄 **JSON → ingestión inicial**
* ❌ **CSV → evitar en Big Data serio**

---

## 🏗️ 3. Data Lake vs DW vs Lakehouse

### 📊 Comparativa clave

| Característica | Data Lake | Data Warehouse | Lakehouse |
| -------------- | --------- | -------------- | --------- |
| Datos          | Crudos    | Estructurados  | Ambos     |
| Costo          | Bajo      | Alto           | Medio     |
| Rendimiento    | Bajo      | Alto           | Alto      |
| Flexibilidad   | Alta      | Baja           | Alta      |
| ACID           | No        | Sí             | Sí        |

---

### 🧠 Regla mental

* **Data Lake → guardar TODO**
* **DW → analizar**
* **Lakehouse → ambos (moderno)**

---

## ⚙️ 4. Frameworks de Procesamiento

### 📊 Spark vs Flink

| Feature    | Spark         | Flink          |
| ---------- | ------------- | -------------- |
| Modelo     | Micro-batch   | Streaming real |
| Latencia   | Segundos      | Milisegundos   |
| Facilidad  | Alta          | Media          |
| Ecosistema | 🔥 Muy grande | Medio          |

---

### 🧠 Regla rápida

* 📊 **Spark → estándar general**
* ⚡ **Flink → tiempo real crítico**

---

## ☁️ 5. Arquitecturas

| Tipo       | Ventajas          | Desventajas           |
| ---------- | ----------------- | --------------------- |
| Cloud      | Escalable, barato | dependencia proveedor |
| On-Premise | Control total     | caro                  |
| Híbrido    | flexible          | complejo              |

---

### 🧠 Regla

👉 Hoy TODO tiende a **Cloud + Lakehouse**

---

## ⚡ 6. Conceptos de Escalabilidad

| Concepto       | Qué hace          |
| -------------- | ----------------- |
| Sharding       | divide datos      |
| Replicación    | copia datos       |
| Load balancing | distribuye carga  |
| Cluster        | grupo de máquinas |
| Nodo           | una máquina       |

---

## 🧩 7. Particionamiento

| Tipo  | Cuándo usar           |
| ----- | --------------------- |
| Rango | fechas                |
| Hash  | distribución uniforme |
| Lista | categorías            |

👉 **Clave:** evitar “hot partitions”

---

## 🔄 8. Pipeline Big Data (flujo típico)

```
Fuente → Kafka → Data Lake (Parquet) → Spark → DW → BI
```

---

## 🛠️ 9. Stack típico Big Data

| Herramienta    | Rol                 |
| -------------- | ------------------- |
| Apache Kafka   | Ingesta streaming   |
| Apache Spark   | Procesamiento       |
| Apache Flink   | Streaming avanzado  |
| Apache Airflow | Orquestación        |
| Trino          | SQL sobre lake      |
| Elasticsearch  | Búsqueda            |
| Databricks     | Plataforma completa |

---

## 🧠 10. Batch vs Streaming (ultra resumido)

| Tipo      | Latencia | Uso              |
| --------- | -------- | ---------------- |
| Batch     | alta     | reportes         |
| Streaming | baja     | fraude, realtime |

---

## 💡 11. Reglas de Oro (EXAM / ENTREVISTA)

### 🚨 Formatos

* Parquet = **default**
* Avro = **Kafka**
* JSON = **entrada**

---

### 🚨 Arquitectura

* Moderno = **Lakehouse + Spark**
* Antiguo = Hadoop + MapReduce

---

### 🚨 Procesamiento

* General → Spark
* Tiempo real crítico → Flink

---

### 🚨 Diseño

* Separar:

  * raw
  * staging
  * analytics

---

## 🧪 12. Casos de Uso (rápido)

| Industria      | Uso             |
| -------------- | --------------- |
| Banca          | fraude          |
| Retail         | recomendaciones |
| Telco          | análisis red    |
| IoT            | sensores        |
| Redes sociales | tendencias      |

---

## 🎯 13. Ejemplo mental completo

👉 E-commerce moderno:

```
Usuarios → Kafka → Data Lake (Parquet)
        → Spark → Lakehouse (Delta)
        → BI / ML
```

---

## 🧠 14. Resumen ULTRA corto

* Big Data = **distribuido**
* Parquet = **formato rey**
* Spark = **motor principal**
* Kafka = **ingestión**
* Lakehouse = **arquitectura moderna**

