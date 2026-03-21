
## Syllabus: Bases de Datos para IA
**Profesor:** Carlos César Sánchez Coronel

### Bloque 1: Arquitectura y Estructuras de Datos

#### Semana 1: El Universo del Dato y la Infraestructura Física
* **La Materia Prima:** Diferenciación técnica entre Dato (valor atómico), Información (dato con significado) y Metadata (datos que describen otros datos).
* **Hardware y Rendimiento:** Funcionamiento del **Buffer Pool** en RAM y la latencia de **I/O** en discos SSD/NVMe.
* **Redes de Almacenamiento:** Uso de **SAN** (Storage Area Network) para alta disponibilidad y redundancia masiva.


#### Semana 2: Modelamiento y Arquitectura Conceptual
* **Ciclo de Modelamiento:** Niveles de abstracción que incluyen el Modelo Entidad-Relación (Conceptual), Lógico y Físico.
* **Diseño Lógico:** Definición de Llaves Primarias (PK) y Llaves Foráneas (FK) para soportar las reglas de negocio.
* **Diseño Físico:** Implementación real en el SGBD incluyendo tipos de datos (VARCHAR, INT), índices y particionamiento.

#### Semana 3: Normalización e Integridad Técnica
* **Control de Redundancia:** Aplicación de las Formas Normales para evitar anomalías: **1FN** (Atomicidad), **2FN** (Dependencia Total) y **3FN** (Dependencia Transitiva).
* **Restricciones de Integridad:** Uso de Constraints (Check/Unique) y mantenimiento de la integridad referencial para evitar "hijos huérfanos".

#### Semana 4: Algoritmos de Búsqueda y Estructuras de Persistencia (SQL & NoSQL)
* **Complejidad Computacional:** Introducción a la notación Big O aplicada a bases de datos ($O(1)$, $O(\log n)$, $O(n)$).
* **Estructuras SQL:**
    * **B-Trees:** Algoritmo de árboles balanceados para índices estándar con complejidad $O(\log n)$.
    * **Hash Indexing:** Algoritmo para búsquedas de igualdad exacta logrando $O(1)$.
* **Estructuras NoSQL:**
    * **LSM-Trees (Log-Structured Merge-Tree):** Optimización de alta velocidad de escritura para motores documentales y de columna.
    * **Inverted Index:** Algoritmo de indexación para búsqueda de texto masivo, fundamental en motores de búsqueda.
    * **Adjacency Lists:** Estructuras para el recorrido de nodos en bases de datos de Grafos.
---

### Bloque 2: Manipulación y Lógica de Servidor

#### Semana 5: Dominio del Lenguaje SQL (DML)
* **Motor de Operatividad:** Uso de SELECT para proyección y filtrado; INSERT, UPDATE y DELETE para transacciones.
* **Lógica de Conjuntos (Joins):** Operaciones de INNER JOIN (Intersección $A \cap B$), LEFT/RIGHT JOIN para preservar integridad de tablas maestras, y FULL OUTER JOIN (Unión $A \cup B$).

#### Semana 6: Análisis Avanzado y Transformación
* **Agregaciones:** Resumen de volúmenes de datos con GROUP BY y funciones estadísticas (SUM, AVG, COUNT, MIN, MAX).
* **Window Functions:** Cálculos complejos (OVER, RANK, ROW_NUMBER, LEAD, LAG) esenciales para analítica de series temporales e IA.
* **Limpieza de Datos:** Operaciones de transformación de texto (RegEx), gestión de zonas horarias y aritmética de intervalos.

#### Semana 7: Programación en el Servidor
* **Encapsulamiento:** Vistas (Views) para simplificar el acceso al usuario final y añadir capas de seguridad.
* **Lógica Interna:** Funciones y Procedimientos Almacenados para automatizar procesos pesados directamente en el motor.
* **Automatismos:** Triggers (Disparadores) ejecutados ante eventos DML, fundamentales para logs de auditoría y consistencia en tiempo real.

---

### Bloque 3: Ingeniería, Big Data e IA

#### Semana 8: Captura de Datos y Extracción Masiva
* **Fuentes Externas:** Técnicas de Web Scraping para alimentar LLMs y uso de Wayback Machine para recuperar datos históricos.
* **Tiempo Real:** Captura de cambios mediante CDC (Change Data Capture) e intercambio de datos por APIs.

#### Semana 9: Pipelines y Paradigmas de Procesamiento
* **Arquitecturas de Movimiento:** Evolución y comparativa entre ETL (Extract, Transform, Load) y ELT (Extract, Load, Transform).
* **Procesamiento de Eventos:** Diferencias entre Batch (lotes) para procesos masivos y Streaming (tiempo real) para detección de fraude.

#### Semana 10: Big Data y Formatos de Alto Rendimiento
* **Almacenamiento Optimizado:** Ventajas del formato columnar Apache Parquet para procesos de entrenamiento masivo de modelos.
* **Arquitecturas Modernas:** Diferencia entre Data Lakes y Data Warehouses; introducción al modelo Lakehouse con Databricks.

#### Semana 11: NoSQL y Ecosistema Cloud
* **Especialización NoSQL:** Motores documentales (MongoDB), Clave-Valor (Redis), Grafos (Neo4j) y Columnares (Cassandra).
* **Infraestructura Cloud:** Comparativa On-Premise vs. Cloud Native (PaaS) en AWS, Azure y GCP.

#### Semana 12: Bases de Datos Vectoriales y RAG
* **Vectores e IA:** Conceptos de Embeddings (representaciones numéricas) y búsqueda por Similitud (ANN) mediante métricas de distancia.
* **Flujo RAG:** Patrón Retrieval Augmented Generation para conectar LLMs con datos reales y evitar alucinaciones.
* **Orquestación:** Uso de LangChain y LlamaIndex con motores como Pinecone o Milvus.

#### Semana 13: Despliegue y Alta Disponibilidad
* **Contenedores:** Despliegue en Docker con Volúmenes persistentes y orquestación con Kubernetes.
* **Estrategias HA:** Implementación de Réplicas de Lectura para distribuir carga y mecanismos de Failover automático.

#### Semana 14: Gestión de Desastres y Arquitecturas Híbridas
* **Resiliencia:** Backups, Snapshots y Point-in-Time Recovery mediante logs de transacciones.
* **Persistencia Políglota:** Sincronización entre motores SQL y Vectoriales mediante CDC para mantener la paridad de información en flujos de IA.

---
