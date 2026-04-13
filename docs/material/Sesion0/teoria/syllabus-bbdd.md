## Syllabus: Bases de Datos para IA

### Bloque 1: Fundamentos y Modelado de Datos

#### Semana 1: Ecosistema de Datos

* **El Universo del Dato:** Diferenciación técnica entre Dato (valor atómico), Información (dato con significado) y Metadata (datos que describen otros datos).
* **Infraestructura Física:** Funcionamiento del **Buffer Pool** en RAM, latencia de **I/O** en SSD/NVMe y uso de **SAN** (Storage Area Network) para alta disponibilidad.
* **Paradigmas de Almacenamiento:** Introducción a SQL vs. NoSQL y su relación con los casos de uso en IA.

#### Semana 2: Modelamiento

* **Ciclo de Abstracción:** Niveles Conceptual (Entidad-Relación), Lógico y Físico.
* **Diseño Lógico:** Llaves Primarias (PK) y Llaves Foráneas (FK).
* **Diseño Físico:** Tipos de datos, índices básicos y particionamiento.

#### Semana 3: Normalización

* **Formas Normales:** 1FN, 2FN, 3FN.
* **Integridad:** Constraints y reglas referenciales.
* **Desnormalización:** Optimización para analítica e IA.

---

### Bloque 2: Manipulación y Lógica de Servidor

#### Semana 4: SQL - Parte 1

* **DML Básico:** `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
* **Filtros y Proyecciones.**
* **Joins Fundamentales:** `INNER`, `LEFT`, `RIGHT`, `FULL`.

#### Semana 5: SQL - Parte 2

* **Agregaciones:** `GROUP BY`, funciones (`SUM`, `AVG`, etc.).
* **Subqueries:** Escalares, correlacionadas.
* **CTEs:** `WITH` para modularizar consultas.

#### Semana 6: SQL - Parte 3

* **Window Functions:** `OVER`, `RANK`, `ROW_NUMBER`, `LAG`, `LEAD`.
* **Feature Engineering en SQL:** Series temporales, cohortes.
* **Transformaciones Avanzadas:** manejo de fechas, texto y limpieza.

#### Semana 7: Optimización de Consultas

* **Big O en bases de datos.**
* **Índices:** B-Tree y Hash.
* **Planes de Ejecución:** `EXPLAIN`, detección de cuellos de botella.

#### Semana 8: Programación en el Servidor

* **Views:** abstracción y seguridad.
* **Stored Procedures y Functions.**
* **Triggers:** automatización y auditoría.

---

### Bloque 3: Ingeniería de Datos, Big Data e IA

#### Semana 9: ETL

* **ETL vs ELT.**
* **Ingesta de Datos:** APIs, archivos, scraping.
* **Batch vs Streaming.**

#### Semana 10: Big Data

* **Formatos:** Parquet.
* **Arquitecturas:** Data Lake, Data Warehouse, Lakehouse.
* **Procesamiento:** Spark, MapReduce.

#### Semana 11: Cloud Database

* **On-Premise vs Cloud.**
* **PaaS vs Serverless.**
* **Proveedores:** AWS, Azure, GCP.

#### Semana 12: NoSQL

* **Tipos:** Documental, Key-Value, Grafos, Columnar.
* **Casos de uso en IA.**
* **Persistencia políglota.**

#### Semana 13: Base de Datos Vectorial

* **Embeddings y Similaridad (ANN).**
* **Índices:** HNSW.
* **Motores:** Pinecone, Milvus, pgvector.

#### Semana 14: Arquitecturas Híbridas

* **RAG (Retrieval Augmented Generation).**
* **Orquestación:** LangChain, LlamaIndex.
* **Sincronización:** CDC entre SQL y vectoriales.

