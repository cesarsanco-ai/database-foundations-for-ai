
## Sesión 11

# NoSQL

## Tipos, Casos de Uso, Integración con IA y Despliegue en la Nube

**Autor:** Carlos César Sánchez Coronel
**Fecha:** 2026

---

# Introducción a NoSQL

Las bases de datos relacionales (SQL) han dominado el mundo de los datos durante décadas. Sin embargo, con la llegada de aplicaciones web a gran escala, big data y la necesidad de flexibilidad, surgieron las bases de datos **NoSQL (Not Only SQL)**. Estas bases de datos están diseñadas para:

* Escalar horizontalmente.
* Manejar datos no estructurados o semiestructurados.
* Ofrecer alta disponibilidad y rendimiento.

---

## ¿Por qué NoSQL?

Limitaciones de SQL en escenarios modernos:

* Escalabilidad vertical costosa (más CPU/RAM en un solo servidor).
* Esquema rígido (schema-on-write) que dificulta cambios frecuentes.
* Dificultad para manejar grandes volúmenes de datos no estructurados.
* Rendimiento limitado en operaciones masivas de lectura/escritura.

---

## Teorema CAP

En sistemas distribuidos, **no se puede garantizar simultáneamente**:

* **Consistencia (C)**: Todos los nodos ven los mismos datos al mismo tiempo.
* **Disponibilidad (A)**: Cada petición recibe respuesta (aunque no sea la más reciente).
* **Tolerancia a Particiones (P)**: El sistema sigue funcionando aunque se pierda comunicación entre nodos.

Sistemas NoSQL típicos eligen **dos de tres**:

* **CP**: Consistencia + tolerancia a particiones (ej. HBase, MongoDB configurable).
* **AP**: Disponibilidad + tolerancia a particiones (ej. Cassandra, CouchDB).
* **CA**: Consistencia + disponibilidad (sistemas tradicionales, no toleran particiones).

---

## Tipos de Bases de Datos NoSQL

| Tipo                | Ejemplos              | Características principales                |
| ------------------- | --------------------- | ------------------------------------------ |
| Documental          | MongoDB, Couchbase    | Documentos JSON/BSON, esquema flexible     |
| Clave-Valor         | Redis, DynamoDB       | Almacenamiento simple de pares clave-valor |
| Grafos              | Neo4j, Amazon Neptune | Modela nodos y relaciones                  |
| Columnar            | Cassandra, HBase      | Datos organizados por columnas, escalables |
| Motores de Búsqueda | Elasticsearch, Solr   | Indexación y búsqueda de texto completo    |

---

# Bases de Datos Documentales: MongoDB

* **Modelo**: Documentos BSON (JSON binario), jerárquicos, con arrays y subdocumentos.
* **Consultas**: Basadas en JSON; permite inserción, actualización, eliminación y agregaciones.
* **Escalabilidad**: Sharding horizontal + replicación para alta disponibilidad.
* **Casos de uso**: Catálogos, CMS, sesiones de usuario, almacenamiento de embeddings para IA.

---

# Bases de Datos Clave-Valor: Redis

* **Modelo**: Clave-valor en memoria, estructuras complejas (listas, sets, hashes).
* **Persistencia**: RDB (snapshots) o AOF (registro de operaciones).
* **Escalabilidad**: Replicación maestro-esclavo y Redis Cluster.
* **Casos de uso**: Caché, sesiones de usuario, rate limiting, colas de tareas, almacenamiento de embeddings para IA.

---

# Bases de Datos de Grafos: Neo4j

* **Modelo**: Nodos (entidades), relaciones (con dirección y tipo), propiedades.
* **Lenguaje**: Cypher (consultas orientadas a grafos).
* **Casos de uso**: Recomendación, detección de fraude, redes sociales, gestión de identidades.
* **Integración IA**: Alimenta sistemas de recomendación basados en relaciones.

---

# Bases de Datos Columnares: Apache Cassandra

* **Modelo**: Filas y columnas, clave primaria con clave de partición + clustering columns.
* **Arquitectura**: Peer-to-peer, sin nodo maestro.
* **Consistencia**: Eventual (configurable).
* **Escalabilidad**: Lineal, alta disponibilidad.
* **Casos de uso**: IoT, series temporales, mensajería, big data transaccional.

---

# Motores de Búsqueda: Elasticsearch

* **Modelo**: Documentos JSON indexados, shard y replicas, inverted index para búsquedas rápidas.
* **Consultas**: API RESTful, agregaciones y búsquedas de texto completo.
* **Casos de uso**: Búsqueda web, análisis de logs, métricas, búsqueda semántica.

---

# Comparativa SQL vs NoSQL

| Característica | SQL                    | NoSQL                                         |
| -------------- | ---------------------- | --------------------------------------------- |
| Esquema        | Fijo (schema-on-write) | Flexible (schema-on-read)                     |
| Transacciones  | ACID                   | BASE / eventual consistency                   |
| Escalabilidad  | Vertical               | Horizontal                                    |
| Consultas      | Estructuradas (JOINs)  | Según tipo, no joins en general               |
| Casos de uso   | Transaccional          | Big Data, tiempo real, datos no estructurados |

---

# Integración con Inteligencia Artificial

* **Embeddings**: Guardados en MongoDB, Elasticsearch o PostgreSQL con pgvector.
* **Caché de resultados/Modelos**: Redis reduce latencia en inferencias de IA.
* **Estado de conversaciones**: Redis mantiene contexto en chatbots.
* **Recomendaciones con grafos**: Neo4j alimenta modelos basados en relaciones.

---

# Infraestructura en la Nube para NoSQL

| Proveedor Cloud | Servicios NoSQL principales                            |
| --------------- | ------------------------------------------------------ |
| AWS             | DynamoDB, DocumentDB, ElastiCache, Neptune, OpenSearch |
| Azure           | Cosmos DB, Redis Cache, Table Storage                  |
| GCP             | Firestore, Bigtable, Memorystore (Redis)               |

**Estrategias de despliegue**:

* Serverless: Escalado automático, sin gestión de servidores (DynamoDB, Cosmos DB).
* Instancias gestionadas: Control sobre configuración (ElastiCache, DocumentDB).
* On-premise vs Cloud: Basado en costo, latencia, cumplimiento.

---

# Dimensionamiento y Costos

* DynamoDB: por capacidad de lectura/escritura (RCU/WCU) y almacenamiento.
* Redis ElastiCache: desde $0.016/hora para instancias pequeñas.
* Cosmos DB: aprox. $0.008/hora por 100 RU/s.

---

# Glosario

* **NoSQL**: Not Only SQL, bases de datos no relacionales.
* **Teorema CAP**: Consistencia, Disponibilidad, Tolerancia a Particiones.
* **Sharding**: Particionamiento horizontal de datos.
* **Replicación**: Copia de datos en varios nodos.
* **Embedding**: Representación vectorial de datos para IA.
* **Latencia**: Tiempo de respuesta.

