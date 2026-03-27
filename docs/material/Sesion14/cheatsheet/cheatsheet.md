---
layout: default
---

# Cheatsheet: Despliegue y Alta disponibilidad
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-14](../../../sesiones/sesion-14.md)

---

## 1. Métricas Fundamentales de HA

| Métrica | Definición | Fórmula | Objetivo |
| :--- | :--- | :--- | :--- |
| **Disponibilidad** | Tiempo operativo / tiempo total | `A = Activo / (Activo + Inactivo)` | 99.9% (3 nueves) a 99.999% (5 nueves) |
| **MTBF** | Tiempo promedio entre fallos | `MTBF = TiempoTotal / N°Fallos` | Maximizar |
| **MTTR** | Tiempo promedio de reparación | `MTTR = TiempoInactividad / N°Fallos` | Minimizar |
| **RPO** | Pérdida de datos máxima aceptable | — | Segundos (crítico) a horas (no crítico) |
| **RTO** | Tiempo máximo para restaurar servicio | — | Segundos a horas |

---

## 2. Teorema CAP (Brewer)

En sistema distribuido, solo se pueden cumplir **2 de 3**:

| Propiedad | Significado |
| :--- | :--- |
| **C** = Consistency | Todos los nodos ven los mismos datos al mismo tiempo |
| **A** = Availability | Toda solicitud recibe respuesta (no error) |
| **P** = Partition Tolerance | Sistema sigue funcionando aunque la red falle |

**Elección fundamental**:
- **CP**: Prioriza consistencia → PostgreSQL, MongoDB (por defecto)
- **AP**: Prioriza disponibilidad → Cassandra, DynamoDB

---

## 3. Alta Disponibilidad por Tipo de Base de Datos

### 3.1 SQL (Relacionales)

| Arquitectura | Mecanismo | RPO | RTO | Trade-off |
| :--- | :--- | :--- | :--- | :--- |
| **Replicación Síncrona** | Escrituras confirman en primario + réplica | **0** | Segundos-minutos | Mayor latencia; CP |
| **Replicación Asíncrona** | Réplica se actualiza con retraso | >0 (minutos) | Minutos | Mayor disponibilidad; AP |
| **Clúster Shared Storage** | Almacenamiento compartido (SAN) | 0 | Minutos | Storage = SPOF potencial |
| **Distribuido (Raft/Paxos)** | Consenso por mayoría | 0 | Segundos | Mayor latencia; complejidad |

**Protocolos de consenso**:
- **Raft**: Elección de líder por votación → CockroachDB, YugabyteDB, Patroni
- **Paxos**: Consenso clásico → Google Spanner, Amazon Aurora

**On-Premise HA SQL**:
| Componente | Estrategia |
| :--- | :--- |
| Infraestructura | Racks separados, fuentes duales, switches redundantes |
| Almacenamiento | SAN con controladores redundantes o DRBD |
| Orquestación | Patroni (PostgreSQL), Orchestrator (MySQL), Pacemaker |
| Red | NIC bonding, rutas redundantes |

**Cloud Gestionado SQL**:
| Servicio | Arquitectura HA | Garantías |
| :--- | :--- | :--- |
| AWS RDS Multi-AZ | Réplica síncrona en otra AZ | RPO=0, RTO=60-120s, SLA 99.95% |
| AWS Aurora | Almacenamiento 3 AZs (6 réplicas, requiere 4) | RPO=0, failover ~30s, SLA 99.99% |
| Azure SQL MI | Failover groups con réplica síncrona | RPO=0, RTO configurable |
| GCP Cloud SQL HA | Réplica síncrona cross-zone | RPO=0, failover automático |

---

### 3.2 NoSQL

| Tipo | Sistema | Arquitectura HA | Consistencia |
| :--- | :--- | :--- | :--- |
| **Clave-Valor** | Redis | Sentinel (3 instancias) o Cluster | AP; eventual |
| **Documentos** | MongoDB | Replica set (3+ nodos) con elección automática | CP configurable |
| **Columnar** | Cassandra | Factor replicación 3; quórum configurable | AP; eventual |
| **Grafos** | Neo4j | Causal Cluster (Raft) | CP |

**Modelos de replicación NoSQL**:

| Modelo | Descripción | Ventaja | Desventaja |
| :--- | :--- | :--- | :--- |
| **Líder-Seguidor** | Un primario escrituras | Simple | SPOF para escrituras |
| **Multilíder** | Múltiples escrituras | Mayor disponibilidad | Conflictos (LWW, CRDTs) |
| **Quórum** | Mayoría confirma operación | Consistencia configurable | Mayor latencia |
| **Sin líder** | Todos aceptan escrituras | Máxima resiliencia | Consistencia eventual |

**On-Premise NoSQL**:
| Sistema | Configuración HA |
| :--- | :--- |
| Cassandra | RF=3 en 3 racks diferentes; consistencia QUORUM |
| MongoDB | Replica set 3 nodos (primario + 2 secundarios) en racks distintos |
| Redis | Sentinel con 3 instancias; failover automático |

**Cloud Gestionado NoSQL**:
| Servicio | HA | SLA |
| :--- | :--- | :--- |
| DynamoDB | 3 AZs automático; replicación síncrona | 99.999% |
| MongoDB Atlas | Multi-AZ con replica set; failover automático | 99.95-99.995% |
| ElastiCache Redis | Multi-AZ con réplica; failover <2min | 99.9% |
| Cosmos DB | Distribución global; consistencia configurable | 99.99-99.999% |

---

### 3.3 Bases de Datos Vectoriales

| Sistema | Arquitectura HA | Desafíos Específicos |
| :--- | :--- | :--- |
| **Milvus** | Clúster con coordinadores (standby), nodos de datos/consulta/índice; replicación de segmentos | Reconstrucción de índices HNSW/IVF costosa |
| **Qdrant** | Clúster con sharding + replicación; consenso Raft para metadatos | Índices en memoria; failover no instantáneo |
| **pgvector** | Hereda HA de PostgreSQL (replicación, Patroni) | Índices vectoriales replicados vía WAL |
| **Pinecone** | Servicio gestionado multi-AZ; índices distribuidos | Failover automático; no expone configuración |
| **Weaviate** | Cloud con almacenamiento S3; recuperación automática | Tiempo de recuperación proporcional al tamaño |

**Desafíos HA en vectoriales**:

| Desafío | Estrategia |
| :--- | :--- |
| Reconstrucción de índices | Almacenamiento persistente del índice; checkpointing |
| Consistencia de embeddings | Versionado; reindexación asíncrona |
| Latencia de failover | Réplicas cálidas (warm replicas) con índice cargado |
| Consistencia eventual | *Read-your-writes* dirigiendo al primario |

---

## 4. Modelos Híbridos On-Premise + Cloud

| Patrón | Descripción | RPO | RTO | Costo |
| :--- | :--- | :--- | :--- | :--- |
| **Backup & Restore** | Backups on-prem → cloud storage | Horas/días | Días | Bajo |
| **Pilot Light** | Infraestructura mínima en cloud; datos replicados | Minutos/horas | Minutos | Medio |
| **Warm Standby** | Réplica activa en cloud; failover automático | Segundos (asínc) o 0 (sínc) | Minutos | Medio-Alto |
| **Active-Active** | Escrituras en ambos entornos | 0 | 0 (inmediato) | Alto |

**Sincronización híbrida**:

| Mecanismo | Uso |
| :--- | :--- |
| **CDC (Debezium + Kafka)** | Captura cambios del WAL/binlog → cloud en tiempo real |
| **Replicación nativa** | PostgreSQL logical replication cross-region |
| **AWS DMS** | Migración/replicación continua on-prem ↔ cloud |

**Requisitos de conectividad**:
- VPN o Direct Connect / ExpressRoute (enlace privado, baja latencia)
- VPC Peering para multicloud

---

## 5. Comparativa Rápida por Tipo

| Aspecto | SQL | NoSQL | Vectorial |
| :--- | :--- | :--- | :--- |
| **Consistencia fuerte** | Nativa (ACID) | Configurable | Limitada |
| **Failover automático** | Sí (con orquestación) | Sí (nativo) | Parcial |
| **Replicación multirregional** | Sí (con latencia) | Sí (diseñado) | Emergente |
| **RPO = 0** | Sí (síncrona) | Limitado (requiere quórum) | Depende del motor |
| **Escalabilidad horizontal escritura** | Limitada | Nativa | Limitada |
| **Tiempo failover típico** | 30-120s | 10-60s | Segundos-minutos |

---

## 6. On-Premise vs. Cloud: Cuadro Rápido

| Aspecto | On-Premise | Cloud Gestionado |
| :--- | :--- | :--- |
| **Control** | Total | Limitado (proveedor gestiona infraestructura) |
| **Tiempo implementación HA** | Semanas | Minutos |
| **SLA** | Interno | 99.9% - 99.999% con compensación |
| **Costo** | CAPEX alto | OPEX por uso |
| **Complejidad operativa** | Alta | Baja |
| **DR nativo** | Requiere segundo centro de datos | Multi-región integrado |

---

## 7. Fórmulas y Reglas Prácticas

### Regla del Quórum
Para `n` nodos, se necesita al menos:
```
mayoría = ⌊n/2⌋ + 1
```
para formar quórum y evitar *split-brain*.

### Conversión de Disponibilidad a Tiempo de Inactividad Anual

| Nivel | Disponibilidad | Tiempo inactividad/año |
| :--- | :--- | :--- |
| 2 nueves | 99% | 3.65 días |
| 3 nueves | 99.9% | 8.76 horas |
| 4 nueves | 99.99% | 52.56 minutos |
| 5 nueves | 99.999% | 5.26 minutos |
| 6 nueves | 99.9999% | 31.5 segundos |

### Relación RPO / RTO
```
RPO ≤ RTO (la pérdida de datos no puede exceder el tiempo de recuperación)
```

---

## 8. Checklist de HA por Capa

| Capa | Verificación |
| :--- | :--- |
| **Infraestructura** | ☐ Fuentes de alimentación redundantes ☐ Racks separados ☐ Climatización redundante |
| **Red** | ☐ NIC bonding ☐ Switches redundantes ☐ Rutas de red alternativas |
| **Almacenamiento** | ☐ RAID apropiado ☐ Controladores redundantes ☐ Backups regulares |
| **Base de Datos** | ☐ Replicación configurada ☐ Failover automático ☐ Monitoreo de estado |
| **Orquestación** | ☐ Detección de fallos ☐ Promoción automática ☐ Fencing (evitar split-brain) |
| **Disaster Recovery** | ☐ RPO/RTO definidos ☐ Pruebas de failover regulares ☐ Documentación de procedimientos |
| **Monitoreo** | ☐ Alertas de disponibilidad ☐ Métricas de latencia ☐ Logs centralizados |

---

## 9. Comandos y Configuraciones Rápidas

### PostgreSQL (Patroni)
```yaml
# Configuración básica patroni.yml
scope: postgres-cluster
restapi:
  listen: 0.0.0.0:8008
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    synchronous_mode: true  # RPO=0
```

### MongoDB Replica Set
```javascript
// Iniciar replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "node1:27017", priority: 2 },
    { _id: 1, host: "node2:27017", priority: 1 },
    { _id: 2, host: "node3:27017", priority: 1, arbiterOnly: true }
  ]
})

// Verificar estado
rs.status()
```

### Cassandra Consistencia
```cql
-- Escritura con quórum
CONSISTENCY QUORUM;
INSERT INTO users (id, name) VALUES (1, 'Alice');

-- Lectura con quórum
CONSISTENCY QUORUM;
SELECT * FROM users WHERE id = 1;
```

### Milvus Replicación
```python
# Crear colección con factor de replicación
from pymilvus import Collection, CollectionSchema, FieldSchema

collection = Collection(
    name="embeddings",
    schema=schema,
    properties={"replication_factor": 3}  # RF=3
)
```

---

## 10. Resumen: Elegir la Estrategia de HA

| Escenario | Recomendación |
| :--- | :--- |
| **Datos financieros, RPO=0 obligatorio** | SQL con replicación síncrona + quórum; cloud: Aurora Multi-AZ |
| **Alta escalabilidad escritura, tolera consistencia eventual** | Cassandra, DynamoDB con factor replicación 3 |
| **Cache/sesiones, tolera pérdida** | Redis Sentinel o ElastiCache Multi-AZ |
| **IA/RAG, embeddings masivos** | Milvus cluster con RF≥2 o Pinecone/Pinecone Cloud |
| **Disaster Recovery económico** | Pilot Light en cloud con CDC |
| **Latencia mínima + HA** | Warm standby cloud con replicación asíncrona |
| **Global, multirregional** | Active-Active (Cosmos DB, CockroachDB, YugabyteDB) |

---

**Principio Fundamental**: No existe HA perfecta; toda arquitectura implica *trade-offs* entre consistencia, disponibilidad, latencia y costo. La estrategia correcta depende de los requisitos de negocio, no solo de consideraciones técnicas.