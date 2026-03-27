# Despliegue y Alta Disponibilidad 


## 1. Fundamentos Teóricos del Despliegue de Bases de Datos

### 1.1. Definición y Alcance

El despliegue de sistemas de bases de datos constituye el conjunto de decisiones arquitectónicas, operativas y de infraestructura que determinan cómo una instancia o clúster de base de datos es instalado, configurado, ejecutado y puesto a disposición de las aplicaciones consumidoras. Este proceso trasciende la mera instalación de software para abarcar dimensiones críticas como la disponibilidad, la escalabilidad, la durabilidad y la recuperación ante fallos.

La teoría del despliegue de bases de datos se fundamenta en tres pilares:

1. **Modelo de consistencia**: Define las garantías que el sistema ofrece respecto a la visibilidad de las escrituras concurrentes.
2. **Arquitectura de distribución**: Determina cómo los datos se particionan y replican entre nodos.
3. **Modelo de fallos**: Establece las suposiciones sobre tipos de fallos (caída de nodos, particiones de red, corrupción de datos) y los mecanismos para tolerarlos.

### 1.2. Taxonomía de Modelos de Despliegue

| Modelo | Caracterización | Fundamentos Teóricos |
| :--- | :--- | :--- |
| **Despliegue Monolítico** | Una sola instancia que gestiona todos los datos y operaciones. | Simplicidad operativa; punto único de fallo (SPOF). Apropiado para cargas de trabajo pequeñas o entornos de desarrollo. |
| **Despliegue Clúster** | Múltiples nodos que operan como una unidad coordinada, compartiendo la carga y/o replicando datos. | Basado en sistemas distribuidos. Elimina SPOF mediante redundancia. Requiere protocolos de consenso. |
| **Despliegue Multirregional** | Nodos distribuidos en múltiples zonas geográficas o centros de datos. | Proporciona resiliencia ante desastres regionales. Introduce complejidad de latencia y consistencia global. |

---

## 2. Teoría de Alta Disponibilidad (HA)

### 2.1. Definición y Métricas Fundamentales

La **alta disponibilidad** es la capacidad de un sistema de permanecer operativo y accesible durante períodos prolongados, minimizando el tiempo de inactividad no planificado.

| Métrica | Definición | Fórmula | Objetivo en HA |
| :--- | :--- | :--- | :--- |
| **Disponibilidad (Availability)** | Proporción de tiempo que el sistema está operativo. | `A = TiempoActivo / (TiempoActivo + TiempoInactivo)` | 99.9% (tres nueves) a 99.999% (cinco nueves) |
| **MTBF (Mean Time Between Failures)** | Tiempo promedio entre fallos consecutivos. | `MTBF = TiempoTotalOperación / NúmeroFallos` | Maximizar |
| **MTTR (Mean Time To Repair)** | Tiempo promedio para restaurar el servicio tras un fallo. | `MTTR = TiempoTotalInactividad / NúmeroFallos` | Minimizar |
| **RPO (Recovery Point Objective)** | Pérdida de datos máxima aceptable medida en tiempo. | — | Segundos a minutos según criticidad |
| **RTO (Recovery Time Objective)** | Tiempo máximo aceptable para restaurar el servicio. | — | Segundos a horas según criticidad |

### 2.2. Teorema CAP y sus Implicaciones para HA

El **Teorema CAP** (Brewer, 2000; formalizado por Gilbert y Lynch, 2002) establece que un sistema distribuido de datos solo puede satisfacer dos de las tres siguientes garantías:

| Propiedad | Definición |
| :--- | :--- |
| **Consistency (C)** | Todos los nodos ven los mismos datos simultáneamente; tras una escritura, todas las lecturas posteriores devuelven ese valor. |
| **Availability (A)** | Cada solicitud recibe una respuesta (no error) aunque algunos nodos hayan caído. |
| **Partition Tolerance (P)** | El sistema continúa operando a pesar de la pérdida arbitraria de mensajes o fallos de red entre nodos. |

**Implicación teórica**: Dado que las particiones de red son inevitables en sistemas distribuidos, los sistemas deben elegir entre **CP** (priorizar consistencia sobre disponibilidad durante particiones) o **AP** (priorizar disponibilidad sobre consistencia).

Esta elección fundamental determina la arquitectura de HA:

- **Sistemas CP**: Durante una partición de red, detienen las escrituras para preservar consistencia (ej. PostgreSQL con replicación síncrona, MongoDB con mayoría de réplicas).
- **Sistemas AP**: Durante una partición, continúan aceptando escrituras, aceptando consistencia eventual (ej. Cassandra, CouchDB).

### 2.3. Teorema PACELC

Extensión del teorema CAP que añade la consideración de latencia: incluso sin particiones (P), los sistemas deben elegir entre consistencia (C) y latencia (L).

| Condición | Trade-off |
| :--- | :--- |
| **Si hay Partición (P)** | Elegir entre Consistencia (C) o Disponibilidad (A) |
| **Else (E), en operación normal** | Elegir entre Consistencia (C) o Latencia (L) |

Este marco teórico explica por qué sistemas como DynamoDB ofrecen consistencia eventual por defecto (menor latencia) pero permiten consistencia fuerte a costa de mayor latencia.

---

## 3. Alta Disponibilidad en Bases de Datos SQL

### 3.1. Fundamentos de HA en Sistemas Relacionales

Los sistemas SQL tradicionalmente priorizan **consistencia fuerte (ACID)** sobre disponibilidad durante fallos. La arquitectura de HA se basa en combinaciones de replicación y failover automático.

### 3.2. Arquitecturas de HA para SQL

| Arquitectura | Descripción | Mecanismos | Trade-offs Teóricos |
| :--- | :--- | :--- | :--- |
| **Replicación Síncrona con Failover Automático** | Uno o más nodos réplica mantienen copia idéntica; las escrituras se confirman solo cuando se confirman en réplica(s). | PostgreSQL (synchronous replication + Patroni), Oracle Data Guard (SYNC), SQL Server Always On (synchronous commit) | **CP system**: Consistencia fuerte; la disponibilidad se sacrifica si la réplica falla o la red se particiona. RPO = 0. |
| **Replicación Asíncrona con Promoción Manual** | Las escrituras se confirman en el primario; las réplicas se actualizan con retraso. | PostgreSQL streaming replication (asíncrono), MySQL replication | **AP system**: Mayor disponibilidad de escritura; posible pérdida de datos (RPO > 0) si el primario falla antes de replicar. |
| **Clúster de Alta Disponibilidad (Shared Storage)** | Múltiples servidores comparten un almacenamiento común (SAN); solo uno activo a la vez. | Windows Failover Cluster, Linux Pacemaker + DRBD | Elimina problema de replicación de datos; el almacenamiento compartido es SPOF potencial. |
| **Clúster Activo-Activo Distribuido** | Múltiples nodos aceptan escrituras simultáneamente con detección de conflictos. | Google Spanner, CockroachDB, YugabyteDB | **CP system con alta disponibilidad**: Usa protocolos de consenso (Paxos, Raft) para lograr consistencia fuerte con tolerancia a fallos. Sacrifica latencia (requiere mayoría de nodos). |

### 3.3. Protocolos de Consenso en SQL Distribuido

Los sistemas SQL distribuidos modernos utilizan protocolos de consenso para coordinar la replicación y el failover:

| Protocolo | Fundamentos | Aplicación |
| :--- | :--- | :--- |
| **Raft** | Elección de líder mediante votación; replicación de log con compromiso mayoritario. | CockroachDB, YugabyteDB, etcd (usado por Patroni) |
| **Paxos** | Protocolo de consenso clásico; variantes como Multi-Paxos para logs replicados. | Google Spanner, Amazon Aurora (implementación propietaria) |

**Teorema fundamental**: En un sistema con `n` nodos, se requiere al menos `⌊n/2⌋ + 1` nodos (mayoría) para formar un quórum que pueda elegir líder y comprometer escrituras, garantizando que no existan dos líderes simultáneos (*split-brain*).

### 3.4. Estrategias de Despliegue On-Premise para SQL HA

| Componente | Estrategia | Justificación Teórica |
| :--- | :--- | :--- |
| **Infraestructura** | Múltiples racks, fuentes de alimentación independientes, switches redundantes. | Eliminar SPOF en capa física; tolerar fallos de componentes individuales. |
| **Almacenamiento** | SAN con controladores redundantes; o almacenamiento local con replicación síncrona (DRBD). | La capa de almacenamiento debe ser altamente disponible para evitar que una falla de disco afecte múltiples nodos. |
| **Red** | NICs en bonding, switches en stack, rutas redundantes. | Tolerar fallos de red sin afectar comunicación entre nodos del clúster. |
| **Orquestación de Failover** | Patroni (PostgreSQL), Orchestrator (MySQL), Clusterware propietario (Oracle Grid). | Automatizar detección de fallos y promoción de réplica con garantías de evitar split-brain mediante fencing. |

### 3.5. Estrategias de Despliegue Cloud para SQL HA

| Servicio Gestionado | Arquitectura de HA | Garantías |
| :--- | :--- | :--- |
| **AWS RDS Multi-AZ** | Réplica síncrona en zona de disponibilidad (AZ) diferente; failover automático a réplica. | RPO = 0 (con sync replication); RTO de 60-120 segundos. |
| **AWS Aurora** | Almacenamiento distribuido en 3 AZs con 6 réplicas; escrituras requieren 4 de 6 confirmaciones. | RPO = 0 (pérdida de datos solo por fallo catastrófico de AZs múltiples); failover en ~30 segundos. |
| **Azure SQL Managed Instance** | Grupos de failover con réplica síncrona en región secundaria; failover automático. | RPO = 0 (modo síncrono); RTO configurable. |
| **Google Cloud SQL (HA)** | Réplica síncrona en zona diferente; failover automático. | RPO = 0; RTO de minutos. |

**Teoría de los "nueves" en cloud**: Los proveedores ofrecen SLA de 99.95% a 99.99% para servicios gestionados, respaldados por acuerdos de compensación. La responsabilidad de disponibilidad se comparte: el proveedor garantiza la infraestructura; el cliente configura correctamente la redundancia.

---

## 4. Alta Disponibilidad en Bases de Datos NoSQL

### 4.1. Fundamentos de HA en Sistemas NoSQL

Los sistemas NoSQL adoptan filosofías de diseño que priorizan la **escalabilidad horizontal** y la **disponibilidad** sobre la consistencia fuerte, siguiendo el paradigma BASE (Basically Available, Soft state, Eventual consistency) en contraposición a ACID.

### 4.2. Arquitecturas de HA por Tipo NoSQL

| Tipo | Ejemplos | Arquitectura de HA | Fundamentos |
| :--- | :--- | :--- | :--- |
| **Clave-Valor** | Redis, Memcached | Redis Sentinel (monitoreo y failover), Redis Cluster (sharding + replicación), ElastiCache (multi-AZ) | **AP system** con failover automático; replicación asíncrona por defecto. Redis Cluster usa gossip protocol para detección de fallos. |
| **Orientado a Documentos** | MongoDB, Couchbase | Replica sets (mínimo 3 nodos), elección automática de primario mediante protocolo de consenso; sharding para escalabilidad horizontal. | **CP system**: Consistencia configurable (write concern). Replica set con mayoría para escrituras. |
| **Orientado a Columnas** | Cassandra, HBase | Replicación con factor configurable (típicamente 3); consistencia configurable por operación (ONE, QUORUM, ALL). | **AP system**: Diseñado para tolerar fallos de nodos y particiones de red. Usa gossip protocol para estado de nodos. |
| **Orientado a Grafos** | Neo4j, Amazon Neptune | Replicación con primario/réplicas; failover automático en configuración de clúster. | Variantes según implementación; Neo4j Causal Cluster usa protocolo Raft para consenso. |

### 4.3. Teoría de Replicación en NoSQL

| Modelo de Replicación | Descripción | Ventajas | Desventajas |
| :--- | :--- | :--- | :--- |
| **Líder-Seguidor (Master-Slave)** | Un nodo primario acepta escrituras; réplicas se actualizan asíncronamente. | Simple; buena para lecturas escalables. | Punto único de fallo para escrituras; posible pérdida de datos. |
| **Multilíder (Multi-Master)** | Múltiples nodos aceptan escrituras; resolución de conflictos necesaria. | Mayor disponibilidad de escritura; tolerancia a fallos de zona. | Complejidad de conflictos (ej. Last Write Wins, resolución por aplicación). |
| **Replicación con Quórum** | Escrituras requieren confirmación de mayoría (N/2+1) de réplicas. | Consistencia configurable; balance entre disponibilidad y consistencia. | Mayor latencia; disponibilidad reducida si fallan más de la mitad de réplicas. |
| **Replicación sin líder (Leaderless)** | Todos los nodos aceptan escrituras; lecturas consultan múltiples nodos y toman la versión más reciente. | Máxima disponibilidad; resiliencia extrema. | Consistencia eventual; necesidad de reparación de lecturas (read repair) y compacciones anti-entropía. |

### 4.4. Estrategias de Despliegue On-Premise para NoSQL HA

| Sistema | Configuración HA On-Premise | Principios |
| :--- | :--- | :--- |
| **Cassandra** | Factor de replicación 3 en 3 racks diferentes; consistencia QUORUM para operaciones críticas. | Tolerancia a fallos de rack completo; sin SPOF. Usa snitches para topología. |
| **MongoDB** | Replica set con 3 nodos (primario, secundario, árbitro o secundario); nodos en racks diferentes. | Elección automática de primario con mayoría; failover en ~10-30 segundos. |
| **Redis** | Redis Sentinel con 3 instancias sentinel; replicación asíncrona con failover automático. | Detección de fallos por consenso de sentinel; promoción automática de réplica. |

### 4.5. Estrategias de Despliegue Cloud para NoSQL HA

| Servicio Gestionado | Arquitectura de HA | Características |
| :--- | :--- | :--- |
| **Amazon DynamoDB** | Almacenamiento distribuido automático en 3 AZs; replicación síncrona por defecto. | **AP system**: Consistencia configurable (eventual o fuerte). SLA 99.999% de disponibilidad. |
| **MongoDB Atlas** | Clusters multi-AZ con replica set; failover automático. | Configurable por nivel; Global Clusters para escrituras multi-región. |
| **Amazon ElastiCache (Redis)** | Multi-AZ con réplica en AZ secundaria; failover automático en < 2 minutos. | Réplica automática; backups automáticos para recuperación. |
| **Azure Cosmos DB** | Distribución global con réplicas configurables; múltiples modelos de consistencia (fuerte, sesión, eventual). | **AP/CP configurable**: Latencia <10ms al 99% para lecturas/escrituras. SLA 99.999%. |

---

## 5. Alta Disponibilidad en Bases de Datos Vectoriales

### 5.1. Fundamentos Teóricos de HA en Sistemas Vectoriales

Las bases de datos vectoriales constituyen una categoría emergente diseñada para almacenar, indexar y consultar *embeddings* de alta dimensionalidad. Su arquitectura de HA presenta desafíos únicos debido a:

1. **Estructuras de índice en memoria**: Índices como HNSW (Hierarchical Navigable Small World) se construyen y mantienen típicamente en RAM para lograr búsquedas de baja latencia.
2. **Alta dimensionalidad**: Los vectores típicamente tienen 384 a 1536 dimensiones, lo que dificulta la compresión y replicación eficiente.
3. **Reconstrucción costosa**: La reconstrucción de índices tras un fallo puede tomar horas para colecciones de miles de millones de vectores.

### 5.2. Arquitecturas de HA para Bases Vectoriales

| Modelo | Descripción | Sistemas que lo implementan | Fundamentos |
| :--- | :--- | :--- | :--- |
| **Replicación Líder-Seguidor** | Nodo primario acepta escrituras e indexación; réplicas se mantienen sincronizadas. | Milvus (replicación de segmentos), pgvector (replicación PostgreSQL nativa) | Similar a SQL; consistencia eventual en réplicas; failover requiere promoción. |
| **Sharding con Replicación** | Datos particionados por clave (hash/rango); cada shard tiene réplicas. | Milvus (sharding por collection), Qdrant (sharding con replicación) | Escalabilidad horizontal + HA; cada shard tiene factor de replicación configurable. |
| **Cloud-Native Distribuido** | Arquitectura de microservicios con almacenamiento separado (S3) e índices en memoria. | Pinecone, Weaviate (versión cloud) | Separación de cómputo (índices) y almacenamiento (objetos). Recuperación mediante recarga de índice desde almacenamiento persistente. |
| **Multi-Región Activo-Activo** | Escrituras aceptadas en múltiples regiones con replicación de índices. | Pinecone (Enterprise), Milvus (Cross-Cluster Search) | Máxima disponibilidad global; complejidad de consistencia vectorial. |

### 5.3. Desafíos Específicos de HA en Sistemas Vectoriales

| Desafío | Descripción | Estrategias de Mitigación |
| :--- | :--- | :--- |
| **Reconstrucción de índices** | La caída de un nodo requiere reconstruir el índice HNSW/IVF, operación costosa en CPU/memoria. | Almacenamiento persistente del índice en disco; checkpointing periódico; uso de índices *recoverable*. |
| **Consistencia de embeddings** | Los vectores son generados por modelos de ML; cambios en el modelo requieren reindexación completa. | Versionado de embeddings; pipelines de reindexación asíncrona; tolerancia a versiones híbridas durante transiciones. |
| **Latencia de failover** | Los índices en memoria no están disponibles durante la recuperación; failover no es instantáneo. | Réplicas cálidas (warm replicas) con índice cargado; failover con tiempo de conmutación en segundos, no milisegundos. |
| **Consistencia eventual vs. búsqueda** | En réplicas asíncronas, un embedding recién insertado puede no estar visible en réplicas durante búsquedas. | Estrategias de *read-your-writes* dirigiendo lecturas al primario; consistencia configurable por operación. |

### 5.4. Estrategias de Despliegue On-Premise para Bases Vectoriales

| Componente | Configuración HA | Fundamentos |
| :--- | :--- | :--- |
| **Milvus** | Clúster distribuido con componentes: Coordinadores (standby), Nodos de datos (con replicación de segmentos), Nodos de índice, Nodos de consulta (con réplicas). | Separación de responsabilidades (microservicios); cada componente puede escalar independientemente; replicación de segmentos con factor 2-3. |
| **Qdrant (Self-Hosted)** | Clúster con sharding y replicación configurable; consenso mediante Raft para metadatos. | Almacenamiento de índices en disco con soporte para memoria; failover automático de shards. |
| **pgvector (PostgreSQL + vector)** | Hereda arquitectura HA de PostgreSQL: replicación síncrona/asíncrona, Patroni, failover automático. | Consistencia ACID; RPO=0 con replicación síncrona; índices vectoriales (HNSW, IVFFlat) replicados como parte del WAL. |

### 5.5. Estrategias de Despliegue Cloud para Bases Vectoriales

| Servicio | Arquitectura de HA | Garantías y Limitaciones |
| :--- | :--- | :--- |
| **Pinecone** | Servicio gestionado con replicación automática multi-AZ; índices distribuidos. | SLA de disponibilidad; failover automático sin intervención. No expone configuración de replicación. |
| **Weaviate Cloud** | Clústeres gestionados con replicación automática; almacenamiento en objetos (S3) para persistencia. | Recuperación automática desde almacenamiento persistente; tiempo de recuperación proporcional al tamaño del índice. |
| **Milvus (Zilliz Cloud)** | Servicio gestionado con configuración de factor de replicación y sharding; multi-AZ. | Consistencia configurable (fuerte/eventual); soporte para multi-región en niveles enterprise. |
| **Amazon OpenSearch Serverless (Vector Engine)** | Servicio gestionado con índices vectoriales distribuidos; replicación automática multi-AZ. | Hereda arquitectura HA de OpenSearch; búsquedas vectoriales con índices HNSW; SLA 99.99%. |

---

## 6. Modelos Híbridos de Despliegue y HA

### 6.1. Fundamentos de la Arquitectura Híbrida para HA

La arquitectura híbrida on-premise/cloud para bases de datos se fundamenta en la optimización del *trade-off* entre control, latencia, costo y resiliencia. Teóricamente, responde a la necesidad de:

1. **Resiliencia ante desastres (Disaster Recovery)**: Extender la HA regional a HA global.
2. **Elasticidad híbrida**: Usar cloud para absorber picos sin sobredimensionar infraestructura on-premise.
3. **Soberanía de datos**: Mantener datos críticos on-premise mientras se aprovechan servicios cloud.

### 6.2. Patrones de HA Híbrida

| Patrón | Descripción | Arquitectura | Consideraciones Teóricas |
| :--- | :--- | :--- | :--- |
| **Pilot Light (Luz Piloto)** | Infraestructura mínima en cloud (réplica de datos) que puede expandirse rápidamente ante fallo on-premise. | Replicación asíncrona (CDC) desde on-prem a cloud; recursos cloud apagados o mínimos en estado normal. | RPO > 0 (minutos/horas); RTO bajo (minutos); costo optimizado. |
| **Warm Standby** | Réplica activa en cloud que puede asumir carga con failover automático o manual. | Replicación síncrona o asíncrona con latencia controlada; cloud ejecuta réplica "caliente". | RPO = 0 con replicación síncrona; RTO segundos a minutos; costo mayor que pilot light. |
| **Active-Active Multirregional** | Escrituras aceptadas tanto on-premise como en cloud simultáneamente. | Replicación bidireccional con resolución de conflictos; coordinación global. | Complejidad extrema; requiere resolución de conflictos (timestamp vectorial, CRDTs). Generalmente evitado excepto para sistemas diseñados desde origen. |
| **Burst to Cloud** | Carga base on-premise; picos de demanda absorvidos por cloud. | Escrituras siempre en on-premise (maestro); lecturas escaladas a réplicas cloud bajo demanda. | Latencia adicional para escrituras; escalabilidad de lecturas elástica. |

### 6.3. Sincronización Híbrida: CDC y Replicación

La sincronización entre entornos on-premise y cloud es crítica para HA híbrida:

| Mecanismo | Descripción | Uso en HA Híbrida |
| :--- | :--- | :--- |
| **Change Data Capture (CDC)** | Captura cambios del log de transacciones (WAL/binlog) y los transmite a cloud. | Debezium + Kafka; AWS DMS; replicación continua para warm standby. |
| **Replicación Nativa** | Mecanismos de replicación propios del motor de base de datos. | PostgreSQL logical replication cross-region; MongoDB Atlas (on-prem to cloud). |
| **ETL Batch** | Movimiento programado de datos (ej. cada hora). | Solo para DR con RPO alto; no adecuado para HA en tiempo real. |

### 6.4. Teoría de Disaster Recovery Híbrido

| Modelo DR | RPO | RTO | Costo | Complejidad |
| :--- | :--- | :--- | :--- | :--- |
| **Backup y Restore** | Horas/Días | Días | Bajo | Baja |
| **Pilot Light** | Minutos/Horas | Minutos | Medio | Media |
| **Warm Standby** | Segundos (asíncrono) o 0 (síncrono) | Minutos | Medio-Alto | Media-Alta |
| **Multi-Site Active-Active** | 0 | 0 (inmediato) | Alto | Alta |

---

## 7. Marco Comparativo Integrado

### 7.1. Comparativa de Capacidades HA por Tipo de Base de Datos

| Capacidad | SQL | NoSQL | Vectorial |
| :--- | :--- | :--- | :--- |
| **Consistencia fuerte** | Nativa (ACID) | Configurable (eventual por defecto) | Limitada (depende del motor) |
| **Failover automático** | Sí (con orquestación) | Sí (nativo en clúster) | Parcial (requiere capa externa) |
| **Replicación multirregional** | Sí (con latencia) | Sí (diseñado para ello) | Emergente (limitada) |
| **RPO = 0** | Sí (replicación síncrona) | Limitado (requiere quórum) | Depende del motor subyacente |
| **Escalabilidad horizontal de escritura** | Limitada (distribuido complejo) | Nativa | Limitada (sharding) |
| **Tiempo de failover típico** | 30-120 segundos | 10-60 segundos | Variable (segundos a minutos) |

### 7.2. Comparativa On-Premise vs. Cloud por Tipo

| Aspecto | SQL On-Premise | SQL Cloud Gestionado | NoSQL On-Premise | NoSQL Cloud Gestionado | Vectorial On-Premise | Vectorial Cloud Gestionado |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Control** | Total | Limitado | Total | Limitado | Total | Limitado |
| **Tiempo de implementación HA** | Semanas | Minutos | Semanas | Minutos | Días | Minutos |
| **SLA garantizado** | Interno | 99.95-99.99% | Interno | 99.9-99.999% | Interno | 99.9-99.99% |
| **Costo de HA** | CAPEX alto | OPEX por uso | CAPEX alto | OPEX por uso | CAPEX alto | OPEX por uso |
| **Complejidad operativa** | Alta | Baja | Alta | Baja | Alta | Baja-Media |
| **Recuperación ante desastre** | Segundo centro de datos | Multi-región nativo | Segundo centro de datos | Multi-región nativo | Backups externos | Multi-región emergente |

---

## 8. Conclusiones Teóricas

1. **La alta disponibilidad es una propiedad arquitectónica, no un añadido**: No puede postergarse ni añadirse sin rediseño fundamental. Requiere decisiones en capa de aplicación, infraestructura, red y almacenamiento.

2. **No existe HA universal**: Cada tipo de base de datos (SQL, NoSQL, Vectorial) ofrece un conjunto diferente de garantías, y la elección debe basarse en los requisitos de consistencia, latencia y tolerancia a fallos de cada carga de trabajo.

3. **El teorema CAP sigue siendo fundamental**: La compensación entre consistencia y disponibilidad es inherente a los sistemas distribuidos. Los sistemas SQL priorizan consistencia; los NoSQL priorizan disponibilidad; los vectoriales están en evolución hacia modelos híbridos.

4. **La hibridación on-premise/cloud redefine HA**: Permite combinar la latencia mínima y control de on-premise con la elasticidad y resiliencia multirregional de cloud, pero introduce complejidad en sincronización y consistencia.

5. **Las bases de datos vectoriales presentan desafíos únicos**: Su dependencia de índices en memoria y la alta dimensionalidad de los datos hacen que la HA sea más compleja que en sistemas tradicionales, requiriendo nuevas estrategias de replicación y recuperación.

6. **La gestión de fallos debe ser proactiva**: La HA efectiva requiere no solo mecanismos de recuperación, sino también monitoreo continuo, pruebas de failover regulares (chaos engineering) y políticas de RPO/RTO alineadas con el negocio.

