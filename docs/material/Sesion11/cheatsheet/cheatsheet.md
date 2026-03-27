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


