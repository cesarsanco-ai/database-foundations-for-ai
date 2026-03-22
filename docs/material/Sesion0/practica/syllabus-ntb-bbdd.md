# Curso de Bases de Datos para Inteligencia Artificial
## Syllabus de Notebooks por Semana

**Carlos César Sánchez Coronel**

2026

---

## Estructura General

Cada semana se trabajará con dos tipos de notebooks:

- **Notebook Conceptual (NB1):** Experimentación con datos dummy, implementación de estructuras, pruebas de rendimiento y simulación de motores de bases de datos.
- **Notebook de Ejercicios (NB2):** Aplicación práctica con herramientas online gratuitas y datasets reales.

---

## Herramientas Online Recomendadas

| Herramienta | Descripción y enlace |
|-------------|---------------------|
| Google Colab | Entorno de notebooks en la nube con GPU gratis. https://colab.research.google.com/ |
| DB-Fiddle | Editor SQL online con PostgreSQL/MySQL. https://www.db-fiddle.com/ |
| SQLiteOnline | Cliente SQLite en navegador. https://sqliteonline.com/ |
| MongoDB Atlas | Base de datos documental gratuita (512 MB). https://www.mongodb.com/atlas |
| Redis Cloud | Base de datos en memoria (30 MB gratis). https://redis.com/try-free/ |
| Neo4j Sandbox | Base de datos de grafos con datasets precargados. https://sandbox.neo4j.com/ |
| Astra DB (Cassandra) | Base de datos columnar gratuita. https://www.datastax.com/products/datastax-astra |
| Elastic Cloud | Motor de búsqueda (prueba gratis). https://www.elastic.co/cloud/ |
| Pinecone | Base de datos vectorial (1 índice gratis). https://www.pinecone.io/ |
| ChromaDB | Base de datos vectorial open-source (local en Colab). https://www.trychroma.com/ |
| Databricks CE | Entorno Big Data con Spark gratis. https://community.cloud.databricks.com/ |
| Great Expectations | Validación de calidad de datos. https://greatexpectations.io/ |
| Looker Studio | Visualización de datos (Google). https://lookerstudio.google.com/ |
| draw.io | Diagramas E-R online. https://app.diagrams.net/ |

---

## Semana 1: Estructuras de Datos para Búsqueda en Bases de Datos (SQL y NoSQL)

### Propósito
Esta semana sienta las bases conceptuales para todo el curso. El objetivo es comprender que **las bases de datos no son cajas mágicas, sino implementaciones sofisticadas de estructuras de datos clásicas**. Mapearemos estas estructuras (árboles, tablas hash, listas, skip lists, LSM-Trees) con motores específicos (PostgreSQL, MySQL, MongoDB, Redis, Cassandra, Neo4j, Elasticsearch) y analizaremos cómo determinan la complejidad algorítmica de las operaciones y, por tanto, el rendimiento.

### Notebook Conceptual (NB1) – Algoritmos y Estructuras: El "Backend" de las Bases de Datos

- **Datos:** Estructuras implementadas en Python y simuladores web.
- **Herramientas:** Google Colab, [B-Tree Visualizer](https://toolkit.whysonil.dev/tools/simulators/btree), [Visualizador de Estructuras de Datos (USFCA)](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html).
- **Actividades:**

  1. **Fundamento: El B-Tree y los Índices SQL:**
     - Usar el B-Tree Visualizer para insertar claves (10, 20, 5, 6, 12, 30, etc.) y observar la división de nodos.
     - Realizar búsquedas y ver el camino resaltado desde la raíz hasta la hoja.
     - Relacionar la profundidad del árbol con la complejidad $O(\log n)$ y el concepto de *fanout*.
     - **Conexión con motores:** PostgreSQL, MySQL, Oracle, SQL Server y MongoDB utilizan B-Tree como estructura de índice por defecto.

  2. **Búsqueda por Hash: La Estructura Detrás de las Claves-Valor:**
     - En Colab, implementar una tabla hash simple con Python (usando listas de listas para manejar colisiones) y medir tiempos de inserción/búsqueda ($O(1)$ promedio).
     - Explicar el concepto de función hash, colisiones y resolución por encadenamiento.
     - **Conexión con motores:**
       - **Redis** es esencialmente una tabla hash gigante en memoria que almacena claves que apuntan a diversas estructuras de datos.
       - Los **índices hash** en PostgreSQL se usan para búsquedas de igualdad exacta.
       - MongoDB soporta **índices hashed**, principalmente para sharding.

  3. **Listas Enlazadas y el Corazón de las Colas:**
     - Implementar una lista doblemente enlazada simple en Python y medir operaciones: inserción al inicio/final ($O(1)$), búsqueda por índice ($O(n)$).
     - **Conexión con motores:**
       - La estructura de datos `List` en Redis es una lista doblemente enlazada, ideal para implementar colas y pilas.
       - Permite operaciones como `LPUSH`, `RPUSH`, `LPOP`, `RPOP` en tiempo constante.

  4. **Skip Lists: La Ingeniería Detrás de los Conjuntos Ordenados:**
     - Usar el visualizador de la Universidad de San Francisco (USFCA) para entender cómo una Skip List combina capas de listas enlazadas para lograr búsqueda $O(\log n)$ promedio.
     - **Conexión con motores:**
       - La estructura de datos `Sorted Set` (ZSET) en Redis se implementa con una Skip List.
       - Esto permite operaciones como `ZRANGE`, `ZRANK`, `ZSCORE` con eficiencia logarítmica.

  5. **LSM-Trees: La Base de las Bases de Datos de Alto Rendimiento en Escritura:**
     - Simular conceptualmente un LSM-Tree en Python: usar una lista en memoria (*memtable*) que al llegar a un tamaño X se "descarga" a un archivo ordenado en disco (*SSTable*).
     - Para la búsqueda, primero buscar en memoria, luego en los SSTables más recientes.
     - Explicar el rol de los **Bloom Filters** para acelerar búsquedas (evitar revisar SSTables donde seguro no está el dato).
     - **Conexión con motores:**
       - **Cassandra**, **HBase**, **RocksDB** y **LevelDB** están basados en LSM-Trees, optimizados para escritura secuencial masiva.

  6. **Índices Invertidos: El Corazón de la Búsqueda de Texto:**
     - Construir un índice invertido simple: dado un conjunto de documentos (oraciones), crear un diccionario donde la clave es una palabra y el valor es la lista de IDs de documentos donde aparece.
     - **Conexión con motores:**
       - **Elasticsearch** y **Solr** (basados en Lucene) utilizan índices invertidos para búsqueda de texto completo.
       - MongoDB ofrece índices de tipo `text` para este propósito.

  7. **Estructuras de Datos en Grafos: Nodos y Relaciones:**
     - Representar un grafo simple en Python: lista de adyacencia (diccionario de listas).
     - Ejecutar una búsqueda de vecinos inmediatos ($O(1)$) y una búsqueda en profundidad limitada.
     - **Conexión con motores:**
       - **Neo4j** está optimizado para almacenar nodos y relaciones (aristas) como estructuras de primer nivel, permitiendo recorridos de grafos (ej. *traversals*) sin costosos JOINs.

- **Actividad de Cierre: Mapeo Motor-Estructura:**
  - Crear una tabla resumen (en el propio notebook) donde los estudiantes completen, para cada motor visto, la estructura de datos principal y la complejidad de sus operaciones clave.

### Notebook de Ejercicios (NB2) – Explorando Motores Reales sin Instalación

- **Objetivo:** Ver cómo la teoría se materializa en motores reales usando herramientas online gratuitas.
- **Actividades:**

  1. **SQL (B-Tree en acción):**
     - Usar [SQLiteOnline](https://sqliteonline.com/).
     - Crear una tabla con 10,000 registros (usando un script en bucle).
     - Ejecutar una búsqueda por un campo sin índice y observar la lentitud (*Table Scan*, $O(n)$).
     - Crear un índice B-Tree en ese campo y volver a ejecutar la búsqueda.
     - Usar `EXPLAIN QUERY PLAN` para verificar el uso del índice.

  2. **Redis (Estructuras en Memoria):**
     - Crear una cuenta gratuita en [Redis Cloud](https://redis.com/try-free/) (30 MB).
     - Conectarse desde Colab usando la librería `redis-py`.
     - Experimentar con las diferentes estructuras:
       - `SET` y `GET` (Strings).
       - `LPUSH`, `RPUSH`, `LPOP` (Lists) y simular una cola.
       - `HSET`, `HGET` (Hashes).
       - `ZADD`, `ZRANGE` (Sorted Sets con Skip Lists).

  3. **MongoDB (Índices y Tipos de Datos):**
     - Crear un cluster gratuito en [MongoDB Atlas](https://www.mongodb.com/atlas) (512 MB).
     - Conectar desde Colab con `pymongo`.
     - Crear una colección con documentos que tengan campos escalares, arrays y coordenadas geoespaciales.
     - Crear y experimentar con diferentes tipos de índices: simple, compuesto, multikey (en arrays), geoespacial.
     - Usar `explain()` para ver cómo MongoDB utiliza (o no) los índices.

  4. **Cassandra (LSM-Tree y Alto Rendimiento en Escritura):**
     - Crear una base de datos gratuita en [Astra DB (DataStax)](https://www.datastax.com/products/datastax-astra).
     - Conectar desde Colab con `cassandra-driver`.
     - Insertar miles de registros y medir la velocidad de escritura (para apreciar la filosofía de diseño de LSM-Tree).
     - Realizar consultas y discutir cómo los Bloom filters ayudan en la lectura.

  5. **Neo4j (Grafos y Relaciones):**
     - Usar [Neo4j Sandbox](https://sandbox.neo4j.com/) (gratuito, con datasets precargados, por ejemplo, "Movies").
     - Ejecutar consultas en Cypher que exploren relaciones:
       - Encontrar actores que trabajaron con un director específico.
       - Encontrar el camino más corto entre dos actores.
     - Visualizar el grafo en la interfaz web de Neo4j.

---

## Semana 2: El Universo del Dato: Infraestructura, Gobernanza y Calidad

### Propósito
Comprender el ciclo de vida del dato desde su generación hasta su explotación, incluyendo el soporte físico (hardware), los roles involucrados en su gestión, y los marcos de gobernanza que garantizan su calidad, privacidad y trazabilidad. Se establecerá la conexión entre estos conceptos y las herramientas prácticas que se usarán en el curso.

### Notebook Conceptual (NB1) – Datos, Hardware y Gobernanza con Python

- **Datos:** Conjuntos simulados en Pandas, logs generados, datasets con errores.
- **Herramientas:** Google Colab, librerías: `pandas`, `timeit`, `psutil` (opcional), `Great Expectations`, `networkx`.
- **Actividades:**

  1. **Dato, Información y Metadata:**
     - Crear un DataFrame con datos crudos (ej. transacciones).
     - Mostrar cómo acceder a la metadata: `.info()`, `.describe()`, `.dtypes`.
     - Discutir la diferencia entre dato (1010), información ("cliente 1010 compró...") y metadata ("la columna 'monto' es entero").
     - Generar logs simulados y explicar su rol en auditoría.

  2. **Simulación de Hardware y Búsqueda:**
     - Recordar brevemente los índices vistos en la Semana 1 (B-Tree, Hash) y su relación con la memoria (RAM) y disco.
     - Usar `timeit` para medir tiempos de acceso a listas (simulando disco) y diccionarios (simulando RAM).
     - Discutir el concepto de *buffer pool*: un caché en RAM de las páginas más usadas. Simular con un diccionario LRU simple.

  3. **Roles en la Gestión de Datos:**
     - Presentar los perfiles: DBA (administración física), Data Engineer (pipelines, modelado), Analista/Data Scientist (explotación).
     - En el notebook, asignar tareas simuladas a cada rol: el DBA crea índices (en SQLite), el Data Engineer construye un pipeline ETL, el Analista genera un dashboard.

  4. **Calidad de Datos con Great Expectations:**
     - Instalar Great Expectations en Colab (`!pip install great_expectations`).
     - Crear un conjunto de datos con errores comunes: valores nulos, duplicados, outliers, formatos incorrectos.
     - Definir expectativas: "columna edad debe ser positiva", "columna email debe tener formato válido", "no debe haber nulos en columna ID".
     - Ejecutar la validación y generar un reporte de Data Docs (en HTML) para visualizar los resultados.

  5. **Linaje de Datos (Data Lineage):**
     - Usar `networkx` para construir un grafo dirigido que represente el flujo de datos: desde fuente (API, base de datos), pasos de transformación (limpieza, agregación), hasta su uso en un modelo de IA.
     - Visualizar el grafo con matplotlib.
     - Discutir la importancia del linaje para auditoría y depuración.

### Notebook de Ejercicios (NB2) – Exploración de Motores, Conectividad y BI

- **Objetivo:** Conocer los principales motores de bases de datos y cómo conectarlos a herramientas de visualización.
- **Herramientas:** Looker Studio, DB-Fiddle, (opcional) Streamlit en Colab.
- **Actividades:**

  1. **Panorama de Motores:**
     - Usar [DB-Fiddle](https://www.db-fiddle.com/) para crear una tabla simple en PostgreSQL y MySQL.
     - Ejecutar las mismas consultas en ambos motores y notar similitudes/diferencias.
     - Investigar brevemente las características de Oracle, SQL Server, PostgreSQL, MySQL (se puede proporcionar una tabla resumen).

  2. **Conectividad y Visualización con Looker Studio:**
     - Subir un conjunto de datos (ej. ventas simuladas) a Google Sheets.
     - Conectar [Looker Studio](https://lookerstudio.google.com/) a esa hoja.
     - Crear un dashboard interactivo con gráficos de barras, líneas de tendencia y filtros.
     - Publicar y compartir el enlace.

  3. **Creación de una interfaz simple con Streamlit (opcional):**
     - En Colab, usar `ngrok` para exponer una app de Streamlit que consulte un DataFrame y muestre resultados.
     - Esto conecta con el concepto de "conectividad" desde Python hacia el usuario final.

---

## Semana 3: Modelamiento Conceptual y Lógico

### Propósito
Diseñar modelos de datos usando diagramas Entidad-Relación y pasarlos a modelo lógico.

### Notebook Conceptual (NB1) – Diagramación con draw.io

- **Herramienta:** https://app.diagrams.net/
- **Actividades:**
  1. Identificar entidades, atributos y relaciones para un caso (ej. biblioteca, ventas).
  2. Crear diagrama E-R.
  3. Convertir a modelo lógico: tablas, PK, FK.
  4. Exportar imagen y documentar.

### Notebook de Ejercicios (NB2) – Implementación en SQL Fiddle

- **Herramienta:** DB-Fiddle (https://www.db-fiddle.com/)
- **Actividades:**
  1. Crear tablas según modelo lógico.
  2. Definir constraints (PK, FK, CHECK).
  3. Insertar datos y probar integridad.

---

## Semana 4: Normalización e Integridad Referencial

### Propósito
Aplicar formas normales para evitar redundancias y garantizar consistencia.

### Notebook Conceptual (NB1) – Desnormalización vs Normalización

- **Herramienta:** Colab + pandas.
- **Actividades:**
  1. Crear DataFrame desnormalizado (repetido, redundante).
  2. Aplicar 1FN, 2FN, 3FN manualmente con operaciones pandas.
  3. Comparar tamaño en memoria y facilidad de consulta.
  4. Discutir trade-offs: rendimiento vs consistencia.

### Notebook de Ejercicios (NB2) – Práctica en SQL Fiddle

- **Actividades:**
  1. Dado un enunciado, crear tablas normalizadas hasta 3FN.
  2. Definir PK y FK.
  3. Intentar insertar datos que violen integridad y observar errores.

---

## Semana 5: SQL DML y JOINs

### Propósito
Dominar operaciones de manipulación y combinación de tablas.

### Notebook Conceptual (NB1) – JOINs con sets en Python

- **Herramienta:** Colab.
- **Actividades:**
  1. Crear dos conjuntos de datos como listas de diccionarios.
  2. Implementar manualmente INNER, LEFT, RIGHT, FULL JOIN usando bucles.
  3. Comparar con `pandas.merge()`.

### Notebook de Ejercicios (NB2) – Práctica en DB-Fiddle

- **Actividades:**
  1. Escribir consultas con diferentes tipos de JOIN.
  2. Resolver ejercicios de múltiples tablas.

---

## Semana 6: Funciones de Ventana y Limpieza

### Propósito
Utilizar window functions y funciones de limpieza para análisis avanzado.

### Notebook Conceptual (NB1) – Window Functions en pandas

- **Herramienta:** Colab.
- **Actividades:**
  1. Generar datos de ventas diarias.
  2. Calcular ranking, LAG, diferencia con mes anterior usando `pandas`.
  3. Comparar con sintaxis SQL equivalente.

### Notebook de Ejercicios (NB2) – Práctica en PostgreSQL (DB-Fiddle)

- **Actividades:**
  1. Usar `ROW_NUMBER()`, `RANK()`, `LAG()`.
  2. Limpiar datos con funciones de texto y fecha.

---

## Semana 7: Vistas, Procedimientos y Triggers

### Propósito
Programar lógica dentro de la base de datos.

### Notebook Conceptual (NB1) – Simulación en Python

- **Herramienta:** Colab.
- **Actividades:**
  1. Simular vista como función que consulta DataFrame.
  2. Simular trigger: función que se ejecuta al modificar datos.

### Notebook de Ejercicios (NB2) – Implementación en Neon (PostgreSQL gratis)

- **Herramienta:** https://neon.tech/ (registro gratis).
- **Actividades:**
  1. Crear vista.
  2. Escribir procedimiento almacenado.
  3. Crear trigger de auditoría.

---

## Semana 8: Captura de Datos: Web Scraping y APIs

### Propósito
Extraer datos de fuentes externas para alimentar bases de datos.

### Notebook Conceptual (NB1) – Scraping en Colab

- **Herramienta:** Colab + `requests`, `BeautifulSoup`.
- **Actividades:**
  1. Scraping de una página simple.
  2. Consumir API pública (JSONPlaceholder).
  3. Guardar resultados en DataFrame y SQLite.

### Notebook de Ejercicios (NB2) – CDC simulado

- **Actividades:**
  1. Leer archivo de logs y detectar cambios.
  2. Actualizar base de datos SQLite con cambios.

---

## Semana 9: Pipelines: ETL vs ELT, Batch vs Streaming

### Propósito
Construir pipelines de datos simples.

### Notebook Conceptual (NB1) – ETL y ELT en Colab

- **Actividades:**
  1. ETL: extraer de API, transformar con pandas, cargar a SQLite.
  2. ELT: cargar crudo a SQLite, transformar con SQL.
  3. Simular streaming con `time.sleep()`.

### Notebook de Ejercicios (NB2) – Batch con datos reales

- **Actividades:**
  1. Usar dataset de ejemplo y procesar por lotes.
  2. Medir tiempos.

---

## Semana 10: Big Data: Databricks Community Edition y PySpark

### Propósito
Introducir entornos de Big Data con Spark usando Databricks gratis.

### Notebook Conceptual (NB1) – Primeros pasos en Databricks CE

- **Herramienta:** https://community.cloud.databricks.com/
- **Actividades:**
  1. Crear cuenta y cluster gratuito (15GB memoria).
  2. Subir datos usando comandos `%sh` o usar datasets precargados (`databricks-datasets/`).
  3. Leer CSV con PySpark y mostrar schema.
  4. Comparar rendimiento con pandas (tiempo en millones de filas).
- **Recursos:**
  - Guía oficial: https://docs.databricks.com/en/getting-started/community-edition.html
  - Datasets incluidos: `%fs ls databricks-datasets/`

### Notebook de Ejercicios (NB2) – Análisis en Databricks

- **Actividades:**
  1. Cargar dataset grande (ej. NYC Taxi desde `databricks-datasets`).
  2. Hacer agregaciones con PySpark SQL.
  3. Crear visualización rápida en notebook.

---

## Semana 11: NoSQL: MongoDB, Redis, Neo4j, Cassandra

### Propósito
Conectar y operar con diferentes motores NoSQL desde Colab.

### Notebook Conceptual (NB1) – Conectando a MongoDB Atlas

- **Herramienta:** MongoDB Atlas (https://www.mongodb.com/atlas) + Colab (`pymongo`).
- **Actividades:**
  1. Crear cluster gratis.
  2. Conectar desde Colab.
  3. Insertar documentos y consultar.

### Notebook de Ejercicios (NB2) – Multi-motor

- **Actividades:**
  1. Redis Cloud: almacenar y leer caché.
  2. Neo4j Sandbox: consultar grafos con `py2neo`.

---

## Semana 12: Bases de Datos Vectoriales y RAG

### Propósito
Implementar búsqueda semántica y flujo RAG con ChromaDB y Pinecone.

### Notebook Conceptual (NB1) – ChromaDB en Colab

- **Herramientas:** Colab, ChromaDB, `sentence-transformers`.
- **Actividades:**
  1. Instalar ChromaDB (`!pip install chromadb`).
  2. Crear embeddings de frases dummy.
  3. Almacenar en Chroma y hacer búsqueda por similitud.
- **Conexiones:**
  - RAG: recuperar documentos relevantes.
  - LangChain como orquestador.

### Notebook de Ejercicios (NB2) – RAG con Pinecone

- **Herramienta:** Pinecone (https://www.pinecone.io/) + LangChain.
- **Actividades:**
  1. Crear índice gratuito en Pinecone.
  2. Cargar documentos, dividir en chunks, incrustar.
  3. Hacer preguntas y generar respuestas con LLM.
- **Comparativa:** ChromaDB (local, gratis) vs Pinecone (nube, gratis limitado).

---

## Semana 13: Docker y Alta Disponibilidad

### Propósito
Desplegar bases de datos en contenedores y entender réplicas.

### Notebook Conceptual (NB1) – Play with Docker

- **Herramienta:** https://labs.play-with-docker.com/
- **Actividades:**
  1. Levantar contenedor PostgreSQL.
  2. Conectar desde otro contenedor.
  3. Simular fallo y reinicio.

### Notebook de Ejercicios (NB2) – Réplicas simuladas

- **Actividades:** En Colab, simular réplica de lectura con hilos.

---

## Semana 14: Gestión de Errores y Arquitecturas Híbridas

### Propósito
Integrar múltiples motores (SQL + NoSQL + Vectorial) en una arquitectura coherente.

### Notebook Conceptual (NB1) – Integración final en Colab

- **Actividades:**
  1. SQLite como transaccional.
  2. ChromaDB para embeddings.
  3. Diccionario Python simulando Redis.
  4. Flujo: insertar en SQLite → disparar actualización de embedding → guardar en Chroma.

### Notebook de Ejercicios (NB2) – Plan de recuperación

- **Actividades:** Simular backup y restore con SQLite.