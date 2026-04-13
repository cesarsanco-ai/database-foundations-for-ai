---
layout: default
---
# Sesion 12: NoSQL y persistencia poliglota para IA

### 1. Logro de la sesion

Evaluar modelos NoSQL (documental, key-value, columnar y grafos), comprender sus trade-offs frente a SQL y disenar estrategias de persistencia poliglota para soluciones de analitica e inteligencia artificial.

---

### 2. Conexion con el syllabus

Semana 12 corresponde a:

- Tipos NoSQL: documental, key-value, grafos, columnar.
- Casos de uso en IA.
- Persistencia poliglota.

---

### 3. Por que surge NoSQL

Limitaciones de sistemas relacionales en ciertos contextos:

- esquemas altamente variables,
- escalado horizontal masivo,
- baja latencia en volumen extremo,
- grafos complejos de relaciones.

NoSQL no reemplaza SQL, lo complementa.

---

### 4. SQL vs NoSQL (visión practica)

SQL destaca en:

- integridad fuerte,
- joins complejos,
- transacciones ACID.

NoSQL destaca en:

- flexibilidad de esquema,
- escalado horizontal,
- modelos de acceso especializados.

---

### 5. Modelo documental

Ejemplos: MongoDB, Couchbase.

Caracteristicas:

- datos en documentos JSON/BSON,
- esquema flexible,
- consultas por campos anidados.

Caso de uso:

perfiles de usuario y eventos semiestructurados.

---

### 6. Modelo key-value

Ejemplos: Redis, DynamoDB (modo KV).

Caracteristicas:

- acceso por clave,
- latencia muy baja,
- ideal para cache y sesiones.

Caso de uso:

almacenar tokens, sesiones, feature flags.

---

### 7. Modelo wide-column / columnar NoSQL

Ejemplos: Cassandra, HBase.

Caracteristicas:

- particion por clave,
- alta escritura distribuida,
- diseño orientado a consultas previstas.

Caso de uso:

telemetria y series de tiempo a gran escala.

---

### 8. Modelo de grafos

Ejemplos: Neo4j, JanusGraph.

Caracteristicas:

- nodos y aristas con propiedades,
- recorridos eficientes de relaciones,
- consultas de caminos y comunidades.

Caso de uso:

fraude, recomendacion, redes sociales, knowledge graphs.

---

### 9. Consistencia y CAP

Teorema CAP en sistemas distribuidos:

- Consistency,
- Availability,
- Partition tolerance.

En presencia de particiones, se balancea consistencia vs disponibilidad.

Conceptos practicos:

- consistencia eventual,
- lecturas/escrituras quorums,
- trade-offs por dominio.

---

### 10. NoSQL en IA

Casos frecuentes:

1. almacenamiento de eventos de inferencia,
2. perfiles enriquecidos por usuario,
3. cache de features online,
4. grafos de conocimiento,
5. historial semiestructurado de prompts/respuestas.

---

### 11. Persistencia poliglota

Es usar mas de un motor segun necesidad.

Ejemplo de arquitectura:

- PostgreSQL para core transaccional,
- Redis para cache de baja latencia,
- MongoDB para documentos flexibles,
- motor vectorial para retrieval semantico.

---

### 12. Seleccion por patrón de acceso

Pregunta clave:

"Como se consultan y actualizan los datos realmente?"

No elegir motor por tendencia, sino por:

- latencia objetivo,
- cardinalidad,
- tipo de consulta,
- costo operativo.

---

### 13. Modelado en NoSQL

A diferencia de SQL, se diseña alrededor de queries esperadas.

Principios:

- desnormalizar donde convenga,
- evitar joins distribuidos costosos,
- precomputar vistas de lectura.

---

### 14. Integridad y calidad en NoSQL

Aunque el esquema sea flexible, se requiere:

- contratos de datos,
- validaciones en ingestion,
- versionado de estructura,
- monitoreo de calidad.

---

### 15. Seguridad en motores NoSQL

Controles minimos:

- autenticacion fuerte,
- autorizacion por coleccion/tabla,
- cifrado de datos,
- redes privadas,
- auditoria de operaciones.

---

### 16. Operacion y observabilidad

Monitorear:

- latencia p95,
- throughput,
- uso de memoria,
- reintentos/errores,
- estado de replicas.

---

### 17. Costos y mantenimiento

Dimensionar costo por:

- almacenamiento,
- replica,
- transferencia,
- operaciones por segundo.

Riesgo tipico:

subestimar costo de indices y replicas.

---

### 18. Migracion y convivencia SQL + NoSQL

Estrategia segura:

1. identificar dominio candidato,
2. pilotear caso acotado,
3. definir sincronizacion,
4. medir impacto,
5. escalar gradualmente.

---

### 19. Errores frecuentes

- usar NoSQL para todo sin justificacion,
- no definir modelo de consistencia,
- copiar esquemas SQL sin adaptar,
- ignorar gobernanza y ownership,
- no planear estrategia de backup.

---

### 20. Caso aplicado

Asistente omnicanal:

- PostgreSQL: usuarios, permisos, facturacion.
- Redis: sesiones activas y cache de contexto.
- MongoDB: historial de conversaciones enriquecidas.
- Vector DB: retrieval semantico de conocimiento.

---

### 21. Mini laboratorio

1. Clasificar 4 casos de uso por tipo de motor.
2. Disenar documento JSON para perfil enriquecido.
3. Proponer esquema de cache en key-value.
4. Definir consulta de grafo para recomendacion.
5. Justificar arquitectura poliglota final.

---

### 22. Checklist de salida

- Distingo los 4 modelos NoSQL.
- Selecciono motor segun patron de acceso.
- Entiendo trade-offs de consistencia.
- Diseño persistencia poliglota razonada.
- Relaciono NoSQL con requerimientos de IA.

---

### 23. Preguntas de autoevaluacion

1. Cuando prefieres documental sobre relacional?
2. Que aporta key-value en latencia?
3. Que riesgo tiene consistencia eventual?
4. Cuando un grafo supera a SQL?
5. Como decides entre un motor o varios?

---

### 24. Referencias recomendadas

1. Kleppmann, Designing Data-Intensive Applications.
2. MongoDB Architecture Guide.
3. Redis documentation.
4. Cassandra data modeling guides.
5. Neo4j graph modeling basics.
