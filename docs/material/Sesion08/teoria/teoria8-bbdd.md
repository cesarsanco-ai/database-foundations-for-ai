
## Sesión 8
# CAPTURA DE DATOS Y EXTRACCIÓN MASIVA
## Fuentes, técnicas y almacenamiento para Big Data e IA

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción a la Captura de Datos

En la era de la inteligencia artificial y el big data, los datos son el nuevo petróleo. Pero antes de que puedan ser refinados (procesados, analizados, usados para entrenar modelos), deben ser **capturados** de sus fuentes originales. Esta sesión se centra en las técnicas y herramientas para extraer datos de diversas fuentes, tanto internas como externas, a pequeña y gran escala. Comprender estos mecanismos es fundamental para cualquier ingeniero de datos, ya que la calidad y la oportunidad de los datos capturados determinan el éxito de los proyectos posteriores.

## Importancia de la Captura de Datos

- **Alimentación de modelos de IA**: Los modelos requieren grandes volúmenes de datos diversos (texto, imágenes, sensores) que deben ser extraídos de la web, APIs, bases de datos, etc.

- **Toma de decisiones en tiempo real**: Aplicaciones como detección de fraude, recomendaciones personalizadas o monitoreo de infraestructura necesitan capturar datos en streaming.

- **Análisis histórico**: Para estudios de tendencias, se necesitan datos pasados (ej. archivos web, logs).

- **Integración de sistemas**: Las empresas modernas tienen decenas de fuentes de datos; capturarlos de forma unificada es un desafío.

## Clasificación de Fuentes de Datos

Podemos clasificar las fuentes según su origen y naturaleza:

- **Internas**: Bases de datos transaccionales (SQL, NoSQL), logs de servidores, archivos corporativos (CSV, Excel), sensores IoT propios.

- **Externas**: Web (páginas públicas), APIs de terceros (redes sociales, clima, finanzas), datos gubernamentales, redes sociales, repositorios públicos (Common Crawl, Wikipedia).

Y según su forma de entrega:

- **Batch**: Los datos se obtienen en bloques periódicos (diario, semanal).

- **Tiempo real (streaming)**: Los datos se generan y capturan continuamente (eventos, sensores).

# Fuentes de Datos Externas y Web Scraping

## Web Scraping

El web scraping es la técnica de extraer información de sitios web de forma automatizada. Es una de las formas más comunes de obtener datos públicos para análisis, investigación o entrenamiento de modelos.

### Casos de Uso

- **Entrenamiento de LLMs**: Modelos como GPT fueron entrenados con grandes corpus extraídos de la web, como Common Crawl (miles de millones de páginas).

- **Monitoreo de precios**: Empresas que rastrean precios de la competencia.

- **Agregadores de noticias**: Recopilan artículos de múltiples fuentes.

- **Investigación académica**: Análisis de redes sociales, tendencias.

### Técnicas y Herramientas

- **Peticiones HTTP**: Usar librerías como `requests` en Python para obtener el HTML.

- **Parsing HTML**: BeautifulSoup, lxml para extraer datos estructurados.

- **Navegación automatizada**: Selenium, Playwright para sitios con JavaScript dinámico.

- **Frameworks de scraping**: Scrapy (Python) para proyectos a gran escala.

- **Respeto de robots.txt y políticas**: Es ético y legal verificar las restricciones del sitio.

### Ejemplo Básico con Python

```python
import requests
from bs4 import BeautifulSoup

url = 'https://quotes.toscrape.com/'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

for quote in soup.select('.quote'):
    text = quote.select_one('.text').get_text()
    author = quote.select_one('.author').get_text()
    print(f'{author}: {text}')
```

### Consideraciones Legales y Éticas

- Respetar el archivo `robots.txt` del sitio.

- No sobrecargar el servidor (usar delays, rate limiting).

- Verificar los términos de servicio; algunos sitios prohíben el scraping.

- Para uso comercial, consultar con un abogado.

### Common Crawl: El Tesoro de la Web

Common Crawl es una organización sin fines de lucro que realiza rastreos masivos de la web y pone los datos a disposición pública. Sus archivos (petabytes de datos) han sido la base para entrenar modelos de lenguaje como GPT, BERT y otros. Los datos están en formato WARC (Web ARChive) y se pueden procesar con herramientas como Apache Spark o Amazon Web Services (AWS) ofrecen estos conjuntos de datos directamente en S3.

## APIs (Application Programming Interfaces)

Las APIs son la forma moderna y estructurada de intercambiar datos entre sistemas. Permiten acceder a datos de servicios como Twitter, Facebook, Google Maps, etc., de manera controlada.

### Tipos de APIs

- **REST**: Basadas en HTTP, usan verbos (GET, POST, PUT, DELETE) y devuelven JSON/XML. Son las más comunes.

- **GraphQL**: Permite al cliente especificar exactamente los campos que necesita.

- **SOAP**: Protocolo más pesado, basado en XML, usado en entornos empresariales.

### Autenticación y Rate Limiting

La mayoría de las APIs requieren una clave de API (API key) o tokens OAuth. Además, imponen límites de tasa (rate limiting) para evitar abusos.

### Ejemplo con Python (API REST)

```python
import requests
import json

api_key = 'tu_api_key'
url = f'https://api.themoviedb.org/3/movie/popular?api_key={api_key}&language=es-ES'
response = requests.get(url)
if response.status_code == 200:
    data = response.json()
    for movie in data['results']:
        print(movie['title'])
else:
    print('Error:', response.status_code)
```

### APIs en Tiempo Real

Algunas APIs ofrecen streaming (WebSockets) para recibir datos en tiempo real, como la API de Twitter (ahora X) para tweets en vivo.

# Captura de Datos en Tiempo Real y Streaming

## Concepto de Streaming

El procesamiento en streaming maneja datos que llegan continuamente, en lugar de lotes. Es esencial para aplicaciones que requieren latencias muy bajas (milisegundos).

### Casos de Uso

- Detección de fraude en transacciones bancarias.

- Monitoreo de sensores IoT (temperatura, presión).

- Análisis de redes sociales en tiempo real.

- Logs de servidores y aplicaciones.

## Change Data Capture (CDC)

CDC es una técnica para capturar cambios en bases de datos (inserciones, actualizaciones, eliminaciones) en tiempo real, sin afectar el rendimiento del sistema transaccional.

### Métodos de CDC

- **Basado en logs**: El motor de BD escribe los cambios en un log transaccional (WAL en PostgreSQL, binlog en MySQL). Herramientas como Debezium leen esos logs y los publican en Kafka.

- **Basado en triggers**: Se crean triggers que insertan los cambios en una tabla de auditoría, que luego es leída. Afecta más el rendimiento.

- **Basado en consultas periódicas**: No es tiempo real, sino batch.

### Debezium y Kafka

Debezium es una plataforma de CDC que se integra con Apache Kafka. Conecta a bases de datos como PostgreSQL, MySQL, MongoDB, etc., y produce mensajes en Kafka por cada cambio. Luego, consumidores pueden procesar esos eventos.

::: center
:::

## Apache Kafka

Kafka es la plataforma de streaming más utilizada. Actúa como un sistema de mensajería distribuido, durable y de alta吞吐量.

- **Tópicos**: Categorías de mensajes.

- **Productores**: Aplicaciones que envían mensajes.

- **Consumidores**: Aplicaciones que leen mensajes.

- **Partitions**: Permiten paralelismo y orden dentro de una partición.

Ejemplo conceptual: Un productor envía tweets a un tópico \"tweets\", y varios consumidores (análisis de sentimiento, almacenamiento) leen de él.

## Otras Plataformas de Streaming

- **AWS Kinesis**: Servicio gestionado de AWS para streaming.

- **Google Pub/Sub**.

- **Apache Pulsar**.

- **RabbitMQ** (más orientado a colas).

# Formatos de Datos y Almacenamiento

Una vez capturados, los datos deben almacenarse de forma eficiente para su posterior procesamiento. La elección del formato y el sistema de almacenamiento es crucial.

## Formatos de Archivo

- **CSV**: Simple, legible, pero ineficiente para grandes volúmenes (sin esquema, no comprimido).

- **JSON**: Muy usado en APIs, flexible pero verboso.

- **Avro**: Formato binario con esquema, ideal para serialización en streaming (Kafka).

- **Parquet**: Formato columnar, altamente comprimido, óptimo para análisis (Spark, Hive).

- **ORC**: Similar a Parquet, usado en Hive.

- **WARC**: Formato de archivo web (Common Crawl).

### Comparativa

::: center
  **Formato**   **Tipo**                  **Compresión**   **Uso típico**
  ------------- ------------------------- ---------------- --------------------------
  CSV           Texto fila                Baja             Intercambio simple
  JSON          Texto semi-estructurado   Media            APIs, documentos
  Avro          Binario fila              Alta             Streaming, serialización
  Parquet       Columnar                  Alta             Análisis, data lakes
  ORC           Columnar                  Alta             Hive, Hadoop
:::

## Almacenamiento: Data Lakes y Data Warehouses

- **Data Lake**: Almacena datos en bruto en formatos nativos (archivos). Ej: Amazon S3, Azure Data Lake, HDFS. Se usa para capturar grandes volúmenes sin transformar.

- **Data Warehouse**: Almacena datos estructurados y procesados (esquema estrella). Ej: Snowflake, BigQuery, Redshift.

En la captura, a menudo se guarda primero en un data lake (raw zone) y luego se transforma para análisis.

## Particionamiento

Para facilitar consultas eficientes, los datos en data lakes se suelen particionar por fecha, región, etc. Ejemplo en S3: `s3://bucket/ventas/year=2025/month=03/day=15/ventas.parquet`

# Procesamiento a Gran Escala

Una vez capturados, los datos masivos necesitan ser procesados para limpieza, transformación y preparación. Aquí entran frameworks de procesamiento distribuido.

## Apache Spark

Spark es el motor más popular para procesamiento de grandes volúmenes (batch y streaming). Ofrece APIs en Python (PySpark), Scala, Java.

- **RDD**: Resilient Distributed Dataset (bajo nivel).

- **DataFrames**: Similar a tablas SQL, con optimizaciones.

- **Spark SQL**: Consultas SQL sobre DataFrames.

- **Structured Streaming**: Procesamiento en tiempo real.

Ejemplo de lectura de un archivo Parquet con PySpark:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("lectura").getOrCreate()
df = spark.read.parquet("s3://bucket/ventas/")
df.filter(df.fecha >= "2025-01-01").groupBy("producto").sum("monto").show()
```

## Apache Flink

Alternativa a Spark especializada en streaming, con menor latencia.

## Hadoop MapReduce

El precursor, pero hoy en día en desuso para nuevos proyectos debido a su complejidad.

## Limpieza y Transformación

Los datos capturados suelen estar sucios: formatos inconsistentes, valores nulos, duplicados. En esta etapa se aplican técnicas como las vistas en la sesión 6 (limpieza con SQL o DataFrames). A gran escala, Spark permite hacerlo de manera distribuida.

# Ejemplo Integrador: Pipeline de Captura de Tweets en Tiempo Real

Imaginemos un sistema que captura tweets sobre un tema, los procesa y los almacena para análisis posterior.

### Arquitectura

1.  **Fuente**: API de Twitter (streaming) envía tweets en tiempo real.

2.  **Ingesta**: Un productor Kafka (escrito en Python) consume la API y publica en un tópico Kafka.

3.  **Almacenamiento**: Un consumidor Spark Structured Streaming lee de Kafka, realiza limpieza (eliminar RT, extraer hashtags) y guarda en Parquet en S3 particionado por fecha.

4.  **Análisis**: Otro proceso batch (Spark) lee los Parquet para análisis de sentimiento o tendencias.

### Código Esquemático

##### Productor Kafka en Python

```python
from kafka import KafkaProducer
import tweepy
import json

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         value_serializer=lambda v: json.dumps(v).encode('utf-8'))

class MyStreamListener(tweepy.Stream):
    def on_data(self, raw_data):
        tweet = json.loads(raw_data)
        # Extraer solo campos relevantes
        data = {
            'id': tweet['id'],
            'text': tweet['text'],
            'user': tweet['user']['screen_name'],
            'created_at': tweet['created_at']
        }
        producer.send('tweets', value=data)
        return True

stream = MyStreamListener(consumer_key, consumer_secret, access_token, access_token_secret)
stream.filter(track=['#datascience'])
```

##### Consumidor Spark Structured Streaming

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, to_date

spark = SparkSession.builder.appName("tweet_processor").getOrCreate()

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "tweets") \
    .load()

# Definir esquema del JSON
schema = "id LONG, text STRING, user STRING, created_at STRING"

tweets = df.select(from_json(col("value").cast("string"), schema).alias("data")).select("data.*")

# Limpieza: quitar RTs y extraer fecha
tweets_clean = tweets.filter(~col("text").startswith("RT")) \
    .withColumn("fecha", to_date("created_at", "EEE MMM dd HH:mm:ss Z yyyy"))

# Guardar en Parquet particionado por fecha
query = tweets_clean.writeStream \
    .format("parquet") \
    .partitionBy("fecha") \
    .option("path", "s3a://bucket/tweets/") \
    .option("checkpointLocation", "/tmp/checkpoint") \
    .trigger(processingTime="5 minutes") \
    .start()

query.awaitTermination()
```

# Ejercicios Resueltos

## Ejercicio 1: Scraping de una Página Web

**Enunciado:** Extraer de la página de citas famosas <http://quotes.toscrape.com> todas las citas, autores y etiquetas, y guardarlas en un archivo CSV.

**Solución:**

```python
import requests
from bs4 import BeautifulSoup
import csv

url = 'http://quotes.toscrape.com'
quotes = []
while url:
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    for quote in soup.select('.quote'):
        text = quote.select_one('.text').get_text()
        author = quote.select_one('.author').get_text()
        tags = [tag.get_text() for tag in quote.select('.tag')]
        quotes.append([text, author, ', '.join(tags)])
    next_btn = soup.select_one('.next a')
    url = next_btn['href'] if next_btn else None

with open('quotes.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['quote', 'author', 'tags'])
    writer.writerows(quotes)
```

## Ejercicio 2: Consumo de API Pública

**Enunciado:** Obtener de la API de JSONPlaceholder (<https://jsonplaceholder.typicode.com/posts>) los posts y guardarlos en un archivo JSON.

**Solución:**

```python
import requests
import json

response = requests.get('https://jsonplaceholder.typicode.com/posts')
posts = response.json()

with open('posts.json', 'w') as f:
    json.dump(posts, f, indent=2)
```

## Ejercicio 3: Simulación de CDC con Triggers en PostgreSQL

**Enunciado:** Crear un trigger en PostgreSQL que, al insertar o actualizar la tabla `clientes`, registre los cambios en una tabla `clientes_audit`. Luego, simular un proceso que lea esos cambios (por ejemplo, con un script Python) para enviarlos a Kafka.

**Solución:**

```sql
-- Tabla de auditoría
CREATE TABLE clientes_audit (
    id SERIAL PRIMARY KEY,
    cliente_id INT,
    nombre_anterior VARCHAR,
    nombre_nuevo VARCHAR,
    email_anterior VARCHAR,
    email_nuevo VARCHAR,
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    operacion CHAR(1) -- I=insert, U=update, D=delete
);

-- Función trigger
CREATE OR REPLACE FUNCTION audit_clientes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO clientes_audit (cliente_id, nombre_nuevo, email_nuevo, operacion)
        VALUES (NEW.id, NEW.nombre, NEW.email, 'I');
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO clientes_audit (cliente_id, nombre_anterior, nombre_nuevo, email_anterior, email_nuevo, operacion)
        VALUES (OLD.id, OLD.nombre, NEW.nombre, OLD.email, NEW.email, 'U');
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO clientes_audit (cliente_id, nombre_anterior, email_anterior, operacion)
        VALUES (OLD.id, OLD.nombre, OLD.email, 'D');
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_audit_clientes
AFTER INSERT OR UPDATE OR DELETE ON clientes
FOR EACH ROW EXECUTE FUNCTION audit_clientes();
```

Luego, un script Python puede leer la tabla de auditoría periódicamente y publicar en Kafka:

```python
import psycopg2
from kafka import KafkaProducer
import json

conn = psycopg2.connect("dbname=test user=postgres")
cur = conn.cursor()
cur.execute("SELECT * FROM clientes_audit WHERE fecha > NOW() - INTERVAL '1 minute'")
rows = cur.fetchall()

producer = KafkaProducer(bootstrap_servers='localhost:9092',
                         value_serializer=lambda v: json.dumps(v).encode('utf-8'))

for row in rows:
    data = {
        'id': row[0],
        'cliente_id': row[1],
        'operacion': row[7],
        'fecha': str(row[6])
    }
    producer.send('clientes_audit', value=data)
```

## Ejercicio 4: Procesamiento Batch con Spark

**Enunciado:** Dado un conjunto de archivos CSV de ventas, cargarlos con Spark, calcular el total de ventas por producto y guardar el resultado en Parquet.

**Solución:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ventas").getOrCreate()

df = spark.read.option("header", "true").csv("ruta/ventas*.csv")
df = df.withColumn("monto", df["cantidad"] * df["precio"])
resultado = df.groupBy("producto").sum("monto").withColumnRenamed("sum(monto)", "total")
resultado.write.parquet("resultado.parquet")
```

# Glosario de Términos

Web Scraping

:   Extracción automatizada de datos de sitios web.

API

:   Interfaz de programación de aplicaciones, permite la comunicación entre sistemas.

CDC (Change Data Capture)

:   Captura de cambios en bases de datos en tiempo real.

Streaming

:   Procesamiento continuo de datos a medida que llegan.

Kafka

:   Plataforma de streaming distribuida.

Debezium

:   Herramienta de CDC que publica cambios en Kafka.

Parquet

:   Formato columnar optimizado para análisis.

Avro

:   Formato binario con esquema, usado en serialización.

Data Lake

:   Repositorio que almacena datos en bruto en su formato original.

Spark

:   Motor de procesamiento distribuido (batch y streaming).

# Referencias {#referencias .unnumbered}

- Common Crawl: <https://commoncrawl.org/>

- Documentación de Apache Kafka: <https://kafka.apache.org/documentation/>

- Debezium: <https://debezium.io/>

- Apache Spark: <https://spark.apache.org/docs/latest/>

- \"Designing Data-Intensive Applications\" - Martin Kleppmann.

- Curso de Web Scraping con Python (Real Python).