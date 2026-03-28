---
layout: default
---

# Cheatsheet: Ecosistema NoSQL
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-11](../../../sesiones/sesion-11.md)

---

## 1️⃣ Conceptos Clave

| Concepto    | Definición                                                                                   |
| ----------- | -------------------------------------------------------------------------------------------- |
| NoSQL       | “Not Only SQL”, bases de datos no relacionales.                                              |
| Teorema CAP | En distribuidos no puedes tener Consistencia + Disponibilidad + Particiones al mismo tiempo. |
| BASE        | Básicamente Disponible, Estado Suave (eventual consistency).                                 |
| Sharding    | Particionamiento horizontal de datos.                                                        |
| Replicación | Copia de datos en varios nodos para alta disponibilidad.                                     |
| Embeddings  | Representación vectorial de datos para IA/ML.                                                |
| Latencia    | Tiempo de respuesta de una operación.                                                        |

---

## 2️⃣ Tipos de NoSQL

| Tipo        | Ejemplos              | Características / Casos de Uso                                            |
| ----------- | --------------------- | ------------------------------------------------------------------------- |
| Documental  | MongoDB, Couchbase    | Documentos JSON/BSON, esquema flexible, CMS, catálogos, datos de usuario. |
| Clave-Valor | Redis, DynamoDB       | Ultra rápido, acceso por clave, caché, sesiones, colas, embeddings.       |
| Grafos      | Neo4j, Amazon Neptune | Nodos y relaciones, recomendaciones, fraude, redes sociales.              |
| Columnar    | Cassandra, HBase      | Columnas anchas, IoT, series temporales, mensajería.                      |
| Búsqueda    | Elasticsearch, Solr   | Texto completo, análisis de logs, métricas, búsqueda web.                 |

---

## 3️⃣ Características de NoSQL

* **Escalabilidad**: horizontal (añadir nodos).
* **Flexibilidad**: esquema dinámico (schema-on-read).
* **Alta disponibilidad**: replicación, tolerancia a fallos.
* **Transacciones**: generalmente eventual consistency (BASE).

---

## 4️⃣ Bases de Datos Populares

| Base          | Tipo                   | Particularidades                                             |
| ------------- | ---------------------- | ------------------------------------------------------------ |
| MongoDB       | Documental             | BSON, sharding, replicación, consultas JSON.                 |
| Redis         | Clave-Valor            | In-memory, estructuras avanzadas, RDB/AOF, clustering.       |
| Neo4j         | Grafo                  | Nodos/relaciones, lenguaje Cypher.                           |
| Cassandra     | Columnar               | Peer-to-peer, alta disponibilidad, consistencia eventual.    |
| Elasticsearch | Búsqueda               | Documentos JSON, inverted index, agregaciones.               |
| DynamoDB      | Clave-Valor/Documental | AWS managed, serverless, escalable.                          |
| Cosmos DB     | Multimodelo            | Azure, document/graph/key-value/column, global distribution. |
| Bigtable      | Columnar               | GCP, similar a HBase, IoT y series temporales.               |

---

## 5️⃣ Lenguajes / Conexión

| Base          | Lenguaje / Conexión        |
| ------------- | -------------------------- |
| MongoDB       | Python: `pymongo`          |
| Redis         | Python: `redis-py`         |
| Neo4j         | Python: `neo4j` (driver)   |
| Cassandra     | Python: `cassandra-driver` |
| Elasticsearch | Python: `elasticsearch-py` |

---

## 6️⃣ Integración con IA

* **Embeddings** → MongoDB, Elasticsearch, Redis.
* **Caché de resultados** → Redis.
* **Estado de conversación** → Redis.
* **Recomendaciones basadas en relaciones** → Neo4j.

---

## 7️⃣ Escalabilidad / Nube

| Estrategia         | Ejemplo Cloud                   |
| ------------------ | ------------------------------- |
| Serverless         | DynamoDB, Cosmos DB             |
| Gestionado         | ElastiCache, DocumentDB         |
| On-Premise / Cloud | Dependiendo de latencia y costo |

---

## 8️⃣ Casos de Uso Rápidos

| Tipo        | Uso típico                                         |
| ----------- | -------------------------------------------------- |
| Documental  | Catálogos, CMS, sesiones, datos de usuario         |
| Clave-Valor | Caché, colas, sesiones, leaderboard, embeddings    |
| Grafos      | Recomendación, detección de fraude, social network |
| Columnar    | IoT, series temporales, mensajería                 |
| Búsqueda    | Logs, búsqueda web, métricas                       |



***


Analogias con SQL

---

### 📊 1. Estructura y Conceptos Básicos

| SQL (PostgreSQL / SQL Server) | MongoDB (NoSQL) | Descripción |
| :--- | :--- | :--- |
| **Database** | **Database** | El contenedor principal. |
| **Table** | **Collection** | Donde guardas los datos (ej: `usuarios`). |
| **Row** (Fila) | **Document** | Un registro individual en formato JSON/BSON. |
| **Column** | **Field** (Campo) | Una clave dentro del JSON (ej: `"pais": "Perú"`). |
| **Primary Key** (`id`) | **`_id`** | Identificador único (por defecto es un `ObjectId`). |
| **View** | **View / Read-only Pipeline** | Una consulta de agregación guardada. |

---

### 🔍 2. Consultas Simples (DML)

| Acción SQL | Filtro en MongoDB Atlas (campo **Filter**) |
| :--- | :--- |
| `SELECT * FROM mensajes` | `{}` (Dejar vacío) |
| `WHERE pais = 'Perú'` | `{ "pais": "Perú" }` |
| `WHERE id > 10` | `{ "_id": { "$gt": 10 } }` |
| `WHERE pais IN ('Perú', 'Chile')` | `{ "pais": { "$in": ["Perú", "Chile"] } }` |
| `WHERE contenido LIKE '%clima%'` | `{ "contenido": { "$regex": "clima", "$options": "i" } }` |
| `WHERE activo = true AND pais = 'Perú'` | `{ "activo": true, "pais": "Perú" }` |



---

### ⚙️ 3. Agregaciones y Operaciones Complejas
Estas se usan en la pestaña **Aggregations** añadiendo "Stages".

| Operación SQL | Stage en MongoDB Atlas | Código de ejemplo (dentro del Stage) |
| :--- | :--- | :--- |
| **WHERE / HAVING** | `$match` | `{ "metadata_ia.sentimiento": "positivo" }` |
| **SELECT** (Alias/Cálculos) | `$project` | `{ "nombre": "$username", "anio": { "$year": "$fecha" } }` |
| **GROUP BY** | `$group` | `{ "_id": "$modalidad", "total": { "$sum": 1 } }` |
| **ORDER BY** | `$sort` | `{ "enviado_en": -1 }` (1: ASC, -1: DESC) |
| **LIMIT** | `$limit` | `5` (Solo el número) |
| **LEFT JOIN** | `$lookup` | `{ "from": "usuarios", "localField": "user_id", ... }` |

---

### 🛠️ 4. Gestión de Errores y Mantenimiento

* **`TRUNCATE TABLE`:** En Atlas usas **"Delete All Documents"** desde los tres puntos `(...)` de la colección.
* **`UNIQUE CONSTRAINT`:** MongoDB lo maneja con el índice `_id`. Si intentas insertar un ID que ya existe, verás el famoso error **`E11000`**.
* **`INSERT INTO ... VALUES (...)`:** En Atlas usas **"Insert Document"**. Si pegas una lista entre corchetes `[ ]`, es un `INSERT` masivo.



