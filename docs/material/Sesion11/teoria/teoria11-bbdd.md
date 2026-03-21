
## Sesión 11
# NoSQL Y ECOSISTEMA CLOUD
## Tipos, casos de uso, integración con IA y despliegue en la nube

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción a NoSQL

Las bases de datos relacionales (SQL) han dominado el mundo de los datos durante décadas. Sin embargo, con la llegada de aplicaciones web a gran escala, big data y la necesidad de flexibilidad, surgieron las bases de datos NoSQL (Not Only SQL). Estas bases de datos están diseñadas para escalar horizontalmente, manejar datos no estructurados o semiestructurados y ofrecer alta disponibilidad.

## ¿Por qué NoSQL?

Las limitaciones de SQL en ciertos escenarios:

- Escalabilidad vertical costosa (más CPU/RAM en un solo servidor).

- Esquema rígido (schema-on-write) dificulta cambios frecuentes.

- Dificultad para manejar grandes volúmenes de datos no estructurados.

- Rendimiento limitado en operaciones de lectura/escritura masivas.

## Teorema CAP

En sistemas distribuidos, es imposible garantizar simultáneamente:

- **Consistencia (C)**: Todos los nodos ven los mismos datos al mismo tiempo.

- **Disponibilidad (A)**: Cada petición recibe una respuesta (aunque no sea la más reciente).

- **Tolerancia a Particiones (P)**: El sistema sigue funcionando aunque se pierda la comunicación entre nodos.

Los sistemas NoSQL eligen dos de tres:

- **CP**: Consistencia y tolerancia a particiones (ej. HBase, MongoDB con configuración).

- **AP**: Disponibilidad y tolerancia a particiones (ej. Cassandra, CouchDB).

- **CA**: Consistencia y disponibilidad (sistemas tradicionales, no toleran particiones).

::: center
:::

## Tipos de Bases de Datos NoSQL

- **Documentales**: Almacenan documentos (JSON, BSON). Ej: MongoDB, Couchbase.

- **Clave-Valor**: Estructura simple de pares clave-valor. Ej: Redis, DynamoDB.

- **Grafos**: Nodos y relaciones. Ej: Neo4j, Amazon Neptune.

- **Columnares**: Almacenan datos en columnas. Ej: Cassandra, HBase.

- **Motores de Búsqueda**: Indexación y búsqueda de texto completo. Ej: Elasticsearch, Solr.

# Bases de Datos Documentales: MongoDB

MongoDB es la base de datos documental más popular. Almacena documentos en formato BSON (binario JSON), con esquema flexible.

## Modelo de Datos

Los documentos son estructuras jerárquicas con campos y valores. Pueden contener arrays y subdocumentos.

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "nombre": "Ana Pérez",
  "email": "ana@mail.com",
  "direccion": {
    "calle": "Av. Siempre Viva",
    "ciudad": "Madrid"
  },
  "pedidos": [
    { "fecha": "2025-01-10", "total": 150 },
    { "fecha": "2025-02-15", "total": 200 }
  ]
}
```

## Consultas

MongoDB usa un lenguaje de consultas basado en JSON.

```python
from pymongo import MongoClient
client = MongoClient('localhost', 27017)
db = client['tienda']
coleccion = db['clientes']
# Insertar
coleccion.insert_one({ "nombre": "Luis", "email": "luis@mail.com" })
# Buscar
result = coleccion.find({ "ciudad": "Madrid" })
for doc in result:
    print(doc)
# Actualizar
coleccion.update_one({ "nombre": "Luis" }, { "$set": { "ciudad": "Barcelona" } })
# Eliminar
coleccion.delete_one({ "nombre": "Luis" })
```

## Índices

MongoDB soporta índices para acelerar consultas. Pueden ser simples, compuestos, geoespaciales, texto.

```python
coleccion.create_index([("ciudad", 1), ("nombre", -1)])
```

## Casos de Uso

- Catálogos de productos (esquema variable).

- Gestión de contenido (blogs, CMS).

- Datos de sesiones de usuarios.

- Integración con IA: almacenar resultados de modelos, datos de entrenamiento.

## Escalabilidad

MongoDB escala horizontalmente mediante **sharding**: los datos se particionan por una clave (shard key) y se distribuyen en varios servidores. También soporta réplicas para alta disponibilidad.

# Bases de Datos Clave-Valor: Redis

Redis es una base de datos en memoria, extremadamente rápida, que soporta diversas estructuras de datos: strings, hashes, listas, sets, sorted sets, etc.

## Características

- **In-memory**: los datos residen en RAM, con persistencia opcional (snapshots, AOF).

- **Baja latencia**: microsegundos.

- **Estructuras ricas**: más allá de simple clave-valor.

- **Operaciones atómicas**: INCR, LPUSH, etc.

## Conexión desde Python

```python
import redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set('clave', 'valor')
print(r.get('clave'))
r.incr('contador')
r.lpush('lista', 'elemento')
```

## Casos de Uso

- **Caché**: almacenar resultados de consultas costosas.

- **Sesiones de usuario**: en aplicaciones web.

- **Rate limiting**: control de peticiones por IP.

- **Colas de tareas** (con listas).

- **Leaderboards** (con sorted sets).

- **IA**: almacenar embeddings, caché de inferencia.

## Persistencia

Redis puede configurarse para persistir:

- **RDB (snapshots)**: volcados periódicos.

- **AOF (Append Only File)**: registro de cada operación.

## Escalabilidad

Redis admite replicación (maestro-esclavo) y clustering (distribución de datos). Redis Cluster particiona automáticamente.

# Bases de Datos de Grafos: Neo4j

Neo4j es la base de datos de grafos líder. Modela datos como nodos, relaciones y propiedades.

## Modelo de Datos

- **Nodos**: entidades (personas, empresas).

- **Relaciones**: conexiones entre nodos (etiquetadas, con dirección).

- **Propiedades**: atributos de nodos y relaciones.

Ejemplo: red social de personas y amistades.

## Consultas con Cypher

Cypher es el lenguaje de consulta de Neo4j, similar a SQL pero orientado a grafos.

```cypher
-- Crear nodos y relaciones
CREATE (p:Persona {nombre: 'Ana', edad: 30})
CREATE (q:Persona {nombre: 'Luis', edad: 32})
CREATE (p)-[:AMIGO_DE]->(q)

-- Consultar amigos de Ana
MATCH (p:Persona {nombre: 'Ana'})-[:AMIGO_DE]->(amigo)
RETURN amigo.nombre
```

## Casos de Uso

- **Sistemas de recomendación** (productos, películas basados en gustos similares).

- **Detección de fraude** (identificar redes de transacciones sospechosas).

- **Redes sociales y análisis de influencia**.

- **Gestión de identidades y accesos**.

## Integración con Python

```python
from neo4j import GraphDatabase
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))
with driver.session() as session:
    result = session.run("MATCH (p:Persona) RETURN p.nombre")
    for record in result:
        print(record["p.nombre"])
```

# Bases de Datos Columnares: Apache Cassandra

Cassandra es una base de datos distribuida diseñada para alta disponibilidad y escalabilidad lineal. Almacena datos en columnas, con un modelo orientado a filas pero con particionamiento.

## Modelo de Datos

En Cassandra, la clave primaria se compone de:

- **Clave de partición**: determina el nodo donde se almacena la fila.

- **Clustering columns**: ordenan las filas dentro de la partición.

Ejemplo de tabla para mensajes:

```sql
CREATE TABLE mensajes (
    usuario_id UUID,
    timestamp TIMESTAMP,
    mensaje TEXT,
    PRIMARY KEY ((usuario_id), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

Las consultas deben incluir la clave de partición para ser eficientes.

## Características

- **Arquitectura peer-to-peer**: sin maestro, todos los nodos iguales.

- **Alta disponibilidad**: sin punto único de fallo.

- **Escalabilidad lineal**: añadir nodos aumenta capacidad.

- **Consistencia eventual** (configurable).

## Conexión desde Python

```python
from cassandra.cluster import Cluster
cluster = Cluster(['localhost'])
session = cluster.connect('mi_keyspace')
rows = session.execute("SELECT * FROM mensajes WHERE usuario_id = %s", [uuid])
for row in rows:
    print(row.timestamp, row.mensaje)
```

## Casos de Uso

- **IoT y series temporales**: ingesta masiva de datos de sensores.

- **Mensajería y chat** (aplicaciones como WhatsApp).

- **Recomendaciones en tiempo real**.

- **Big Data transaccional**.

# Motores de Búsqueda: Elasticsearch

Elasticsearch es un motor de búsqueda y analítica distribuido, basado en Lucene. Indexa documentos JSON y permite búsquedas de texto completo, agregaciones y análisis en tiempo real.

## Conceptos Clave

- **Índice**: colección de documentos.

- **Documento**: unidad de información (JSON).

- **Shard**: partición del índice.

- **Inverted index**: estructura para búsqueda rápida de términos.

## Consultas

Elasticsearch ofrece una API RESTful. Ejemplo con Python:

```python
from elasticsearch import Elasticsearch
es = Elasticsearch(['localhost:9200'])
# Indexar documento
doc = {'autor': 'Cervantes', 'titulo': 'Don Quijote', 'anio': 1605}
es.index(index='libros', body=doc)
# Buscar
resp = es.search(index='libros', body={'query': {'match': {'titulo': 'Quijote'}}})
for hit in resp['hits']['hits']:
    print(hit['_source'])
```

## Casos de Uso

- **Búsqueda en sitios web** (e-commerce, contenido).

- **Análisis de logs** (ELK Stack: Elasticsearch, Logstash, Kibana).

- **Métricas y monitoreo**.

- **Búsqueda semántica** con vectores (plugin de vector).

# Comparativa entre NoSQL y SQL

::: center
  **Característica**   **SQL**                        **NoSQL**
  -------------------- ------------------------------ -----------------------------------------------
  Esquema              Fijo (schema-on-write)         Flexible (schema-on-read)
  Transacciones        ACID                           BASE (eventual consistency)
  Escalabilidad        Vertical                       Horizontal
  Consultas            Estructuradas (JOINs)          Según tipo (no joins en general)
  Casos de uso         Aplicaciones transaccionales   Big Data, tiempo real, datos no estructurados
:::

### Comparativa entre tipos NoSQL {#comparativa-entre-tipos-nosql .unnumbered}

::: center
  **Tipo**      **Ventajas**              **Desventajas**          **Casos típicos**
  ------------- ------------------------- ------------------------ ------------------------
  Documental    Flexible, fácil de usar   Consultas limitadas      Catálogos, CMS
  Clave-Valor   Ultra rápido              Solo acceso por clave    Caché, sesiones
  Grafos        Relaciones complejas      Escalabilidad compleja   Recomendación, fraude
  Columnar      Alta escalabilidad        Modelado complejo        IoT, series temporales
  Búsqueda      Búsqueda texto completo   Consistencia eventual    Logs, búsqueda web
:::

# Integración con Inteligencia Artificial

Las bases de datos NoSQL son fundamentales en pipelines de IA/ML.

## Almacenamiento de Embeddings

Los embeddings (vectores) se pueden almacenar en MongoDB, PostgreSQL con pgvector, o Elasticsearch (plugin de vector). Ejemplo con MongoDB:

```python
from pymongo import MongoClient
import numpy as np
client = MongoClient()
db = client['ia']
coleccion = db['documentos']
embedding = np.random.rand(300).tolist()
coleccion.insert_one({ 'texto': 'ejemplo', 'embedding': embedding })
```

Luego se pueden hacer búsquedas de similitud (coseno) con agregaciones.

## Caché de Modelos y Resultados

Redis se usa para cachear predicciones de modelos, reduciendo latencia.

```python
import redis
import pickle
r = redis.Redis()
modelo = pickle.loads(r.get('modelo_regresion'))
# O cachear resultado de una inferencia
r.setex('user_123_pred', 3600, pickle.dumps(prediccion))
```

## Sesiones de Usuario en IA Conversacional

En chatbots, Redis guarda el estado de la conversación (contexto) para mantener la coherencia.

## Recomendaciones con Grafos

Neo4j puede alimentar modelos de recomendación basados en caminos de usuarios y productos.

# Infraestructura en la Nube para NoSQL

Los principales proveedores cloud ofrecen servicios gestionados de bases de datos NoSQL.

## Amazon Web Services (AWS)

- **DynamoDB**: base de datos clave-valor y documental, totalmente gestionada, escalable, con rendimiento predecible.

- **DocumentDB**: compatible con MongoDB.

- **ElastiCache**: Redis y Memcached gestionados.

- **Neptune**: base de datos de grafos.

- **OpenSearch**: sucesor de Elasticsearch.

## Microsoft Azure

- **Cosmos DB**: base de datos multimodelo (documental, clave-valor, columna, grafos) con distribución global.

- **Redis Cache**.

- **Table Storage** (clave-valor).

## Google Cloud Platform (GCP)

- **Firestore**: documental.

- **Bigtable**: columna ancha (similar a HBase).

- **Memorystore**: Redis.

## Estrategias de Despliegue

- **Serverless**: sin gestión de servidores (DynamoDB, Cosmos DB). Ideal para aplicaciones variables.

- **Instancias gestionadas**: control sobre la configuración (ElastiCache, DocumentDB).

- **On-premise vs Cloud**: decisión basada en costo, latencia, cumplimiento.

## Dimensionamiento y Costos

Ejemplo: DynamoDB cobra por capacidad de lectura/escritura (RCU/WCU) y almacenamiento. Para una aplicación de 1000 lecturas/segundo, se necesitan 1000 RCU (aprox. \$0.065 por hora). Costos aproximados:

- DynamoDB: \$1.25 por millón de escrituras, \$0.25 por millón de lecturas.

- Redis en AWS ElastiCache: desde \$0.016/hora (tipo pequeño).

- Cosmos DB: aprox. \$0.008/hora por cada 100 RU/s.

# Ejercicios Resueltos

## Ejercicio 1: Modelar un catálogo de productos en MongoDB

**Enunciado:** Diseñar un documento para un catálogo de productos que incluya nombre, precio, categoría, etiquetas y especificaciones variables. Insertar un par de ejemplos y consultar productos de una categoría con precio \> 100.

**Solución:**

```python
from pymongo import MongoClient
client = MongoClient()
db = client['tienda']
productos = db['productos']
producto1 = {
    'nombre': 'Laptop Gamer',
    'precio': 1200,
    'categoria': 'Electrónica',
    'etiquetas': ['laptop', 'gaming'],
    'especificaciones': {'ram': '16GB', 'procesador': 'i7'}
}
producto2 = {
    'nombre': 'Mouse',
    'precio': 25,
    'categoria': 'Electrónica',
    'etiquetas': ['mouse', 'inalámbrico'],
    'especificaciones': {'tipo': 'óptico'}
}
productos.insert_many([producto1, producto2])
result = productos.find({'categoria': 'Electrónica', 'precio': {'$gt': 100}})
for p in result:
    print(p['nombre'])
```

## Ejercicio 2: Sistema de votación con Redis

**Enunciado:** Usar Redis para llevar un contador de votos por candidato y mostrar el ranking.

**Solución:**

```python
import redis
r = redis.Redis()
# Votar
r.incr('votos:candidato:A')
r.incr('votos:candidato:B')
r.incr('votos:candidato:A')
# Obtener votos
print(r.get('votos:candidato:A'))
# Ranking (sorted set)
r.zincrby('ranking', 1, 'A')
r.zincrby('ranking', 2, 'B')
r.zincrby('ranking', 1, 'A')
print(r.zrevrange('ranking', 0, -1, withscores=True))
```

## Ejercicio 3: Detección de fraude con grafos

**Enunciado:** En Neo4j, crear un grafo de transacciones entre cuentas. Detectar posibles ciclos de transferencias rápidas (lavado de dinero).

**Solución:**

```cypher
-- Crear cuentas
CREATE (a:Cuenta {id: 'A'}), (b:Cuenta {id: 'B'}), (c:Cuenta {id: 'C'})
CREATE (a)-[:TRANSFIERE {monto: 1000, fecha: '2025-03-01'}]->(b)
CREATE (b)-[:TRANSFIERE {monto: 900, fecha: '2025-03-02'}]->(c)
CREATE (c)-[:TRANSFIERE {monto: 800, fecha: '2025-03-03'}]->(a)
-- Buscar ciclos de longitud 3
MATCH p = (a)-[:TRANSFIERE*3]-(a)
RETURN p
```

## Ejercicio 4: Almacenar logs en Elasticsearch

**Enunciado:** Indexar logs de servidor en Elasticsearch y luego buscar los errores de los últimos 5 minutos.

**Solución:**

```python
from elasticsearch import Elasticsearch
from datetime import datetime, timedelta
es = Elasticsearch()
doc = {
    'timestamp': datetime.now(),
    'nivel': 'ERROR',
    'mensaje': 'Timeout en conexión'
}
es.index(index='logs', body=doc)
# Buscar errores últimos 5 min
hace_5min = datetime.now() - timedelta(minutes=5)
query = {
    'query': {
        'bool': {
            'must': [
                {'match': {'nivel': 'ERROR'}},
                {'range': {'timestamp': {'gte': hace_5min}}}
            ]
        }
    }
}
resp = es.search(index='logs', body=query)
for hit in resp['hits']['hits']:
    print(hit['_source'])
```

# Glosario de Términos

NoSQL

:   Not Only SQL, bases de datos no relacionales.

Teorema CAP

:   Consistencia, Disponibilidad, Tolerancia a Particiones.

MongoDB

:   Base de datos documental.

Redis

:   Base de datos en memoria clave-valor.

Neo4j

:   Base de datos de grafos.

Cassandra

:   Base de datos columnar distribuida.

Elasticsearch

:   Motor de búsqueda y analítica.

DynamoDB

:   Base de datos NoSQL gestionada de AWS.

Cosmos DB

:   Base de datos multimodelo de Azure.

Sharding

:   Particionamiento horizontal de datos.

Replicación

:   Copia de datos en varios nodos.

Embedding

:   Representación vectorial de datos para IA.

Latencia

:   Tiempo de respuesta.

# Referencias {#referencias .unnumbered}

- Sadalage, P. & Fowler, M. (2012). *NoSQL Distilled*. Addison-Wesley.

- Documentación oficial de MongoDB: <https://docs.mongodb.com/>

- Documentación de Redis: <https://redis.io/documentation>

- Documentación de Neo4j: <https://neo4j.com/docs/>

- Documentación de Cassandra: <https://cassandra.apache.org/doc/>

- Documentación de Elasticsearch: <https://www.elastic.co/guide/>

- Amazon DynamoDB: <https://aws.amazon.com/dynamodb/>

- Microsoft Azure Cosmos DB: <https://docs.microsoft.com/azure/cosmos-db/>