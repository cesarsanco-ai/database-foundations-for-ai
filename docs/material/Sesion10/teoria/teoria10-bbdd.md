
## Sesión 10
# BIG DATA Y FORMATOS DE ALTO RENDIMIENTO
## Principios, almacenamiento, procesamiento y casos reales

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción al Big Data

El término **Big Data** se refiere a conjuntos de datos cuyo tamaño, velocidad o variedad exceden la capacidad de las herramientas tradicionales para capturarlos, gestionarlos y procesarlos en un tiempo razonable. No es solo una cuestión de volumen, sino de cómo extraer valor de ellos. En esta sesión exploraremos los conceptos fundamentales, las tecnologías de almacenamiento optimizado, las plataformas de procesamiento distribuido y las arquitecturas modernas como el Lakehouse, con ejemplos concretos de su aplicación en el mundo real, incluyendo el contexto peruano y latinoamericano.

## Las 5 V del Big Data

Inicialmente se definieron 3 V, pero hoy se habla de 5 o más:

1.  **Volumen**: Cantidad masiva de datos generados (terabytes, petabytes, exabytes).

2.  **Velocidad**: Rapidez con la que se generan y deben procesarse (ej. sensores IoT, transacciones bancarias).

3.  **Variedad**: Diferentes formatos (estructurados, semiestructurados, no estructurados).

4.  **Veracidad**: Calidad y confiabilidad de los datos.

5.  **Valor**: Capacidad de transformar los datos en beneficios para el negocio.

## ¿Cuándo una solución es Big Data?

No todo proyecto con muchos datos es Big Data. Se considera Big Data cuando las herramientas convencionales (una sola máquina, bases de datos relacionales) no pueden manejar la carga y se requiere procesamiento distribuido (múltiples nodos). Ejemplos típicos:

- Redes sociales: millones de publicaciones diarias (Twitter, Facebook).

- IoT: millones de sensores enviando lecturas cada segundo.

- Finanzas: transacciones bursátiles en tiempo real.

- Ciencia: secuenciación genética, astronomía.

# Almacenamiento Optimizado para Big Data

En Big Data, los datos no siempre residen en tablas de bases de datos tradicionales. Se utilizan formatos de archivo especialmente diseñados para ser eficientes en espacio y velocidad de lectura.

## Apache Parquet

Parquet es un formato columnar de código abierto. Almacena los datos por columnas en lugar de por filas, lo que permite:

- Lecturas eficientes de subconjuntos de columnas (solo se leen las necesarias).

- Alta compresión (los datos de una misma columna suelen ser similares).

- Esquemas evolutivos (se pueden añadir columnas).

Es el formato nativo de herramientas como Apache Spark, y es ampliamente usado en data lakes (S3, HDFS).

## Apache ORC

Optimized Row Columnar (ORC) es otro formato columnar, similar a Parquet, desarrollado originalmente por Hive. Ofrece características como índices internos y estadísticas para acelerar consultas.

## Apache Avro

Avro es un formato binario orientado a filas, con un esquema definido en JSON. Es ideal para serialización en sistemas de mensajería como Kafka, ya que es compacto y rápido. Soporta evolución de esquema (compatible hacia adelante/atrás).

## Comparativa de Formatos

::: center
  **Formato**   **Tipo**   **Compresión**       **Uso típico**             **Evolución**
  ------------- ---------- -------------------- -------------------------- ---------------
  Parquet       Columnar   Alta (por columna)   Análisis, data lakes       Sí
  ORC           Columnar   Alta                 Hive, Hadoop               Sí
  Avro          Fila       Media                Streaming, serialización   Sí
  JSON          Texto      Baja                 APIs, documentos           No
  CSV           Texto      Baja                 Intercambio simple         No
:::

## Ejemplo: Lectura de Parquet con Spark

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("lectura").getOrCreate()
df = spark.read.parquet("s3://mi-bucket/ventas/")
df.filter(df.fecha >= "2025-01-01").groupBy("producto").sum("monto").show()
```

# Data Lakes, Data Warehouses y Lakehouses

## Data Lake

Un data lake es un repositorio que almacena datos en su formato original (crudo) a gran escala. Suele basarse en sistemas de archivos distribuidos como HDFS o almacenamiento en la nube (S3, ADLS). Ventajas:

- Bajo costo.

- Almacena cualquier tipo de dato (estructurado, semiestructurado, no estructurado).

- Ideal para ciencia de datos y exploración.

Desventajas:

- Puede convertirse en un "data swamp" si no hay gobernanza.

- Las consultas son lentas si no se usan herramientas adecuadas.

## Data Warehouse

Un data warehouse almacena datos estructurados, modelados y optimizados para consultas analíticas (OLAP). Ejemplos: Amazon Redshift, Google BigQuery, Snowflake. Ventajas:

- Alto rendimiento en consultas.

- Gobernanza y calidad de datos.

- Ideal para reporting y BI.

Desventajas:

- Costo más elevado.

- Menos flexible para datos no estructurados.

## Lakehouse

El concepto **Lakehouse** (acuñado por Databricks) busca combinar lo mejor de ambos: almacenamiento tipo data lake (bajo costo, formato abierto) con capacidades de gestión y rendimiento de data warehouse (transacciones ACID, evolución de esquema, optimización de consultas). Se implementa sobre formatos como Delta Lake, Apache Iceberg o Apache Hudi.

::: center
:::

### Delta Lake

Delta Lake es una capa de almacenamiento de código abierto que aporta ACID, versionado de datos (time travel) y manejo de metadatos escalable sobre Parquet. Es el núcleo de Databricks.

### Apache Iceberg

Iceberg es otra tabla de formato abierto diseñada para grandes conjuntos de datos, con soporte en Spark, Flink, Trino, etc. Ofrece evolución de esquema segura y particionado oculto.

## Databricks: La plataforma Lakehouse

Databricks es una plataforma unificada de análisis de datos que integra:

- Motor de procesamiento Apache Spark optimizado (hasta 50x más rápido).

- Capa de almacenamiento Delta Lake.

- Entorno de notebooks colaborativos.

- Soporte para machine learning (MLflow) y SQL.

Es ampliamente usada en empresas de todo el mundo para proyectos de Big Data e IA.

# Procesamiento Distribuido: Frameworks

Para procesar volúmenes masivos de datos, se requiere computación paralela en clústeres de máquinas. Los principales frameworks son:

## Apache Spark

Spark es el motor más popular. Permite procesamiento batch, streaming, SQL, machine learning y graph en un mismo entorno. Utiliza memoria (cuando es posible) para acelerar las operaciones. Ofrece APIs en Scala, Python, Java, R.

- **Resilient Distributed Datasets (RDD)**: estructura fundamental.

- **DataFrames**: abstracción similar a tablas, con optimizaciones.

- **Spark SQL**: consultas SQL sobre DataFrames.

- **Structured Streaming**: procesamiento en tiempo real con API de DataFrames.

## Apache Flink

Flink es un motor de procesamiento de streaming nativo, con baja latencia y capacidad de estado. Es ideal para aplicaciones que requieren resultados en milisegundos.

## Apache Hadoop MapReduce

El precursor, pero hoy en desuso para la mayoría de aplicaciones debido a su alta latencia (escritura intermedia a disco) y complejidad.

## Comparativa Spark vs Flink

::: center
  **Característica**         **Spark**                                          **Flink**
  -------------------------- -------------------------------------------------- ------------------
  Modelo                     Micro-batches (por defecto) o streaming continuo   Streaming nativo
  Latencia                   Segundos (micro-batches)                           Milisegundos
  Procesamiento con estado   Sí (checkpoints)                                   Sí (muy robusto)
  Ecosistema                 Muy amplio (SQL, ML, Graph)                        En crecimiento
  Facilidad de uso           Alta                                               Media
:::

# Arquitecturas Cloud, Híbridas y On-Premise

## Cloud (Nube)

Los proveedores cloud ofrecen servicios gestionados de Big Data:

- **AWS**: EMR (Spark, Hadoop), Kinesis (streaming), Athena (consultas SQL sobre S3), Redshift (data warehouse).

- **Google Cloud**: Dataproc (Spark/Hadoop), Pub/Sub, BigQuery.

- **Azure**: HDInsight, Event Hubs, Synapse.

Ventajas: escalabilidad elástica, pago por uso, menor mantenimiento.

## On-Premise (Local)

La empresa instala y gestiona su propio clúster (Hadoop, Spark) en sus servidores. Ventajas: control total, seguridad (datos no salen de la empresa). Desventajas: alta inversión inicial (CapEx), complejidad operativa.

## Híbrida

Combinación de ambos: algunos datos y procesos en la nube, otros en local. Por ejemplo, un data lake en la nube y procesamiento crítico on-premise, o viceversa.

## Escalabilidad y Concurrencia

En Big Data, la escalabilidad horizontal (añadir más nodos) es clave. La computación paralela permite dividir el trabajo entre muchos servidores. Conceptos importantes:

- **Sharding**: dividir los datos en fragmentos (shards) distribuidos.

- **Replicación**: copiar datos en varios nodos para tolerancia a fallos.

- **Balanceo de carga**: distribuir las consultas entre nodos.

# Casos de Uso Reales

## Redes Sociales (Twitter)

Twitter genera cientos de millones de tweets al día. Utilizan Hadoop y Spark para análisis de tendencias, detección de spam, y recomendaciones. Los datos se almacenan en formatos como Parquet en un data lake.

## Internet de las Cosas (IoT)

Empresas como Petrobras (Brasil) usan IoT en plataformas petroleras. Miles de sensores envían lecturas cada segundo. Los datos se procesan en tiempo real con Kafka y Flink para detectar anomalías y prevenir fallos.

## Finanzas (Bancos)

Un banco en Perú procesa millones de transacciones diarias. Utilizan CDC (Debezium) para capturar cambios en sus bases de datos transaccionales, los envían a Kafka, y luego Spark los transforma y carga en un data warehouse (Redshift) para análisis de fraude y reportes regulatorios.

## Retail (Comercio Electrónico)

Falabella (Chile) usa Big Data para personalización de ofertas. Los clics de los usuarios se capturan en tiempo real, se unen con datos históricos y se alimentan modelos de recomendación (machine learning) en Databricks.

## Telecomunicaciones

Una operadora móvil en Latinoamérica analiza registros de llamadas (CDRs) para optimizar la red y detectar patrones de fraude. Volúmenes de varios terabytes diarios procesados con Spark.

## Big Data en Perú

En Perú, sectores como banca, retail y telecomunicaciones están adoptando Big Data. Ejemplos:

- BCP: uso de data lakes y machine learning para segmentación de clientes.

- Interbank: analítica en tiempo real de transacciones.

- Entel: procesamiento de datos de red para mejorar calidad de servicio.

El crecimiento del cloud (AWS en Lima) está facilitando la adopción.

# Computación Paralela y Concurrente

## Conceptos Fundamentales

- **Paralelismo**: ejecución simultánea de múltiples tareas en diferentes núcleos/máquinas.

- **Concurrencia**: capacidad de manejar múltiples tareas al mismo tiempo (no necesariamente en paralelo).

- **Cluster**: conjunto de máquinas que trabajan juntas.

- **Nodo**: una máquina del clúster.

- **Task**: unidad de trabajo (por ejemplo, procesar una partición de datos).

## Modelos de Particionamiento

Para distribuir datos, se usan técnicas como:

- **Rango**: dividir por rangos de valores (ej. fechas).

- **Hash**: aplicar función hash a una clave.

- **Lista**: por categorías (ej. región).

El particionamiento debe balancear la carga y permitir consultas eficientes.

## Ejemplo: Sharding en Cassandra

En Cassandra, la clave de partición determina el nodo donde se almacena la fila. Las consultas deben incluir la clave de partición para ser eficientes.

# Herramientas y Plataformas Adicionales

## Apache Kafka

Plataforma de streaming para la ingesta y distribución de eventos en tiempo real. Actúa como cola distribuida y duradera. Se integra con Spark, Flink, etc.

## Apache Airflow

Orquestador de workflows. Permite programar y monitorizar pipelines complejos (DAGs). Es el estándar para orquestación ETL/ELT.

## Trino (antes Presto)

Motor de consultas SQL distribuido que puede consultar datos directamente en data lakes (S3, HDFS) en formatos como Parquet. Muy rápido para análisis ad-hoc.

## Apache Hive

Data warehouse sobre Hadoop que traduce SQL a MapReduce o Spark. Hoy menos usado.

## Apache HBase

Base de datos NoSQL columnar sobre HDFS, para acceso aleatorio en tiempo real.

## Elasticsearch

Motor de búsqueda y analítica, usado para logs y datos de texto.

## Tabla Resumen

::: center
  **Herramienta**   **Función principal**
  ----------------- -----------------------------------------------
  Apache Spark      Procesamiento distribuido (batch y streaming)
  Apache Flink      Procesamiento streaming de baja latencia
  Apache Kafka      Mensajería y streaming de eventos
  Apache Airflow    Orquestación de pipelines
  Trino             Consultas SQL sobre data lakes
  Elasticsearch     Búsqueda y análisis de texto
  Databricks        Plataforma unificada Lakehouse
  AWS EMR           Clústeres gestionados de Hadoop/Spark
  Google BigQuery   Data warehouse serverless
:::

# Ejercicios Resueltos

## Ejercicio 1: Elección de formato

**Enunciado:** Una empresa necesita almacenar logs de servidores (texto) para análisis posteriores. ¿Qué formato recomendaría y por qué?

**Solución:** Recomendaría Parquet, ya que permite compresión y consultas eficientes sobre campos específicos (timestamp, nivel de log). Además, se puede usar con Spark para procesar grandes volúmenes. Alternativamente, si se requiere ingestión en tiempo real, podrían usar Avro para Kafka y luego convertirlos a Parquet para almacenamiento.

## Ejercicio 2: Diseño de un data lake

**Enunciado:** Diseñar la estructura de carpetas para un data lake en S3 que almacena ventas diarias, clientes y productos.

**Solución:**

    s3://mi-lake/
      raw/
        ventas/
          year=2025/
            month=03/
              day=15/
                ventas_20250315.csv
              day=16/
                ...
        clientes/
          clientes_20250315.csv
        productos/
          productos.csv
      staging/
        ventas_clean.parquet
        clientes_clean.parquet
      analytics/
        ventas_diarias.parquet

Particionar por fecha facilita consultas posteriores y evolución.

## Ejercicio 3: Contar palabras con Spark

**Enunciado:** Escribir un script en PySpark que cuente las palabras más frecuentes en un conjunto de archivos de texto.

**Solución:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import explode, split, col

spark = SparkSession.builder.appName("wordcount").getOrCreate()
df = spark.read.text("s3://bucket/textos/*.txt")
words = df.select(explode(split(col("value"), "\s+")).alias("word"))
counts = words.groupBy("word").count().orderBy(col("count").desc())
counts.show(10)
```

## Ejercicio 4: Procesamiento streaming con Spark

**Enunciado:** Leer un stream de Kafka con eventos de clics y contar los clics por minuto.

**Solución:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import window, col

spark = SparkSession.builder.appName("streaming").getOrCreate()
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "clics") \
    .load()
# Asumimos que el valor es JSON con timestamp y usuario
from pyspark.sql.types import StructType, StringType, TimestampType
schema = StructType().add("usuario", StringType()).add("timestamp", TimestampType())
clics = df.selectExpr("CAST(value AS STRING)").select(from_json("value", schema).alias("data")).select("data.*")
counts = clics.groupBy(window("timestamp", "1 minute"), col("usuario")).count()
query = counts.writeStream.outputMode("complete").format("console").start()
query.awaitTermination()
```

## Ejercicio 5: Dimensionamiento de clúster

**Enunciado:** Estimar el número de nodos necesarios para procesar 10 TB de datos diarios con Spark, asumiendo que cada nodo tiene 64 GB de RAM y 8 cores, y que el trabajo requiere 4 horas.

**Solución:**

- Supongamos que Spark puede procesar 1 TB por hora por cada 10 cores (depende del tipo de trabajo). Con 8 cores por nodo, 0.8 TB/hora/nodo.

- Para 10 TB en 4 horas, se necesitan 10 / 4 = 2.5 TB/hora.

- Nodos necesarios: 2.5 / 0.8 ≈ 3.125, es decir, 4 nodos.

Este cálculo es muy aproximado; en la práctica se hacen pruebas de rendimiento.

# Glosario de Términos

Big Data

:   Conjuntos de datos masivos que requieren procesamiento distribuido.

Data Lake

:   Repositorio de datos en bruto.

Data Warehouse

:   Almacén de datos estructurados optimizado para consultas.

Lakehouse

:   Arquitectura que combina data lake y data warehouse.

Parquet

:   Formato columnar de alto rendimiento.

ORC

:   Formato columnar similar a Parquet.

Avro

:   Formato binario orientado a filas, usado en streaming.

Spark

:   Motor de procesamiento distribuido.

Flink

:   Motor de procesamiento streaming de baja latencia.

Kafka

:   Plataforma de mensajería distribuida.

CDC

:   Change Data Capture, captura de cambios en bases de datos.

Sharding

:   Particionamiento horizontal de datos.

Clúster

:   Conjunto de máquinas trabajando juntas.

On-Premise

:   Infraestructura local.

Cloud

:   Infraestructura en la nube.

Databricks

:   Plataforma unificada Lakehouse.

Delta Lake

:   Capa de almacenamiento ACID sobre Parquet.

# Referencias {#referencias .unnumbered}

- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly.

- Zaharia, M., et al. (2016). *Apache Spark: A Unified Engine for Big Data Processing*. Communications of the ACM.

- Databricks. *What is a Lakehouse?* <https://databricks.com/blog/2020/01/30/what-is-a-data-lakehouse.html>

- Documentación de Apache Spark: <https://spark.apache.org/docs/latest/>

- Documentación de Apache Kafka: <https://kafka.apache.org/documentation/>

- Amazon Web Services. *Big Data en AWS*. <https://aws.amazon.com/big-data/>

- Artículo: \"Big Data en Perú: estado actual y perspectivas\" (Revista de Ingeniería, 2024).