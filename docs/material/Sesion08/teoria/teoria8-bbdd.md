---
layout: default
---

# PIPELINES DE DATOS

## Introducción

En las sesiones anteriores hemos cubierto desde la captura de datos hasta los paradigmas de procesamiento. Ahora nos adentramos en el corazón de la arquitectura de datos moderna: el \*\*data warehouse\*\* y los \*\*pipelines\*\* que lo alimentan. Esta sesión integra conceptos de almacenamiento, procesamiento batch y streaming, y las diferentes capas que componen un ecosistema analítico. Comprender estas ideas es esencial para diseñar soluciones escalables y eficientes que soporten desde reportes empresariales hasta modelos de inteligencia artificial.

## Objetivos de la sesión

- Definir y diferenciar Data Warehouse, Data Mart, Data Lake y Lakehouse.

- Comprender los procesos ETL y ELT, sus diferencias y cuándo aplicarlos.

- Distinguir entre procesamiento batch y streaming, con herramientas y casos de uso.

- Conocer técnicas de carga incremental y actualización de datos.

- Identificar las capas típicas en un data warehouse (raw, staging, integración, datamart).

- Entender el papel de los metadatos y la gobernanza.

- Explorar la integración con herramientas de visualización y costos asociados.

- Dimensionar almacenamiento y servidores de forma aproximada.

- Analizar arquitecturas híbridas (SQL + NoSQL) y la convivencia de múltiples motores.

- Definir conceptos de pipelines, jobs, tareas y orquestación.

# Fundamentos de Almacenamiento Analítico

## Data Warehouse (DW)

Un data warehouse es un repositorio centralizado que almacena datos estructurados, provenientes de múltiples fuentes, optimizado para consultas analíticas (OLAP). Se caracteriza por:

- **Orientado a temas**: Organizado por entidades de negocio (ventas, clientes, productos).

- **Integrado**: Datos de diversas fuentes se limpian y unifican.

- **No volátil**: Una vez cargados, los datos no se modifican (solo se añaden).

- **Histórico**: Almacena series temporales para análisis de tendencias.

Ejemplos: Amazon Redshift, Google BigQuery, Snowflake, SQL Server Data Warehouse.

## Data Mart

Un data mart es un subconjunto del data warehouse, orientado a un área de negocio específica (ventas, marketing). Puede ser dependiente (se alimenta del DW) o independiente (fuente propia).

## Data Lake

Un data lake almacena datos en bruto en su formato original (estructurados, semiestructurados, no estructurados). Es un repositorio de bajo costo (generalmente en sistemas de archivos como S3, HDFS) que permite guardar grandes volúmenes sin transformar. Ideal para exploración y ciencia de datos.

## Lakehouse

El concepto lakehouse fusiona lo mejor de ambos mundos: la flexibilidad y bajo costo del data lake con las capacidades de gestión y rendimiento del data warehouse. Plataformas como Databricks (sobre Delta Lake) o Apache Iceberg implementan esta arquitectura, permitiendo transacciones ACID, evolución de esquema y consultas eficientes sobre datos en formato abierto (Parquet).

::: center
  **Característica**      **Data Warehouse**       **Data Lake**               **Lakehouse**
  ----------------------- ------------------------ --------------------------- -------------------------
  Formato de datos        Estructurado             Cualquier formato           Abierto (Parquet, etc.)
  Esquema                 Fijo (schema-on-write)   Flexible (schema-on-read)   Evolutivo
  Transacciones ACID      Sí                       Limitado                    Sí
  Rendimiento analítico   Alto                     Bajo/Medio                  Alto
  Costo                   Alto                     Bajo                        Medio
:::

# Procesos ETL vs ELT

## ETL (Extract, Transform, Load)

Es el enfoque tradicional: los datos se extraen de las fuentes, se transforman en un servidor intermedio (staging) y luego se cargan en el data warehouse.

::: center
:::

### Ventajas

- Control total sobre la transformación.

- Adecuado para datos estructurados y limpieza pesada.

- Menor carga en el data warehouse.

### Desventajas

- Requiere infraestructura de transformación separada.

- Mayor latencia (los datos no están disponibles hasta que termina la transformación).

## ELT (Extract, Load, Transform)

Con el poder de cómputo de los data warehouses modernos (Snowflake, BigQuery, Redshift), se popularizó ELT: los datos se cargan primero en el DW y luego se transforman dentro de él.

::: center
:::

### Ventajas

- Simplicidad: menos herramientas, todo en el DW.

- Aprovecha la escalabilidad del DW.

- Mayor agilidad: los datos crudos están disponibles rápidamente.

### Desventajas

- Costo de almacenamiento y cómputo en el DW (puede ser alto).

- No ideal para datos no estructurados (aunque algunos DW soportan JSON).

## Cuándo usar cada uno

- **ETL**: Cuando se necesita transformaciones muy complejas con datos sensibles, o se trabaja con fuentes que no soportan descarga masiva.

- **ELT**: Cuando el DW es lo suficientemente potente, se busca agilidad y se trabaja con grandes volúmenes (cloud).

Hoy en día, la tendencia es ELT en la nube, pero ETL sigue vivo en entornos on-premise o con herramientas como Informatica, Talend.

# Procesamiento Batch vs Streaming

## Batch (Lotes)

El procesamiento batch trabaja con datos acumulados durante un período (horas, días). Se ejecuta en intervalos programados. Es el enfoque tradicional para reportes diarios, facturación, etc.

### Herramientas comunes

- Apache Spark (batch y streaming).

- Hadoop MapReduce.

- Scripts programados con cron, Airflow.

### Ejemplo

El cierre diario de un banco: todas las transacciones del día se procesan en la madrugada para actualizar saldos y generar reportes.

## Streaming (Tiempo Real)

El procesamiento en streaming maneja eventos conforme ocurren, con latencias de milisegundos a segundos. Es crítico para detección de fraude, monitoreo, recomendaciones en tiempo real.

### Herramientas clave

- Apache Kafka (plataforma de streaming).

- Apache Flink (procesamiento con estado).

- Spark Structured Streaming.

- AWS Kinesis, Google Pub/Sub.

### Ejemplo

Detección de fraudes: cada transacción se analiza al instante comparando con patrones históricos.

## Arquitecturas Lambda y Kappa

- **Lambda**: Combina batch y streaming; dos capas que procesan los mismos datos y se unen en la capa de servicio. Compleja de mantener.

- **Kappa**: Todo se procesa como streaming; los datos batch se tratan como un caso particular de streaming (reprocesamiento). Más simple.

::: center
:::

# Técnicas de Carga Incremental

En lugar de recargar todo el data warehouse cada vez, se utilizan cargas incrementales para actualizar solo los datos nuevos o modificados.

## Marcas de agua (Watermarks)

Se guarda la fecha/hora de la última carga y se extraen registros posteriores.

## CDC (Change Data Capture)

Como vimos en la sesión 8, CDC captura cambios en las fuentes y los aplica al DW. Herramientas como Debezium + Kafka permiten streaming de cambios.

## MERGE (UPSERT)

En SQL, se puede usar `MERGE` o `INSERT ON CONFLICT` para actualizar filas existentes e insertar nuevas.

```sql
-- PostgreSQL
INSERT INTO destino (id, valor, fecha_actualizacion)
SELECT id, valor, now() FROM origen
ON CONFLICT (id) DO UPDATE SET valor = EXCLUDED.valor, fecha_actualizacion = EXCLUDED.fecha_actualizacion;
```

## Ejemplo: Carga incremental diaria

1.  Obtener la fecha máxima del DW: `SELECT MAX(fecha) FROM ventas_dw;`

2.  Extraer de la fuente: `SELECT * FROM ventas_source WHERE fecha > :ultima_fecha;`

3.  Cargar en el DW.

# Capas en un Data Warehouse

Un DW bien diseñado suele organizarse en capas (layers) que facilitan el mantenimiento y la comprensión.

## Capa Raw (Bronze)

Almacena los datos tal como llegan de las fuentes, sin transformar. Puede estar en un data lake o en tablas staging. Sirve como respaldo y para reprocesamiento.

## Capa Staging (Silver)

Datos limpios, validados y unificados. Se aplican transformaciones ligeras (tipos de datos, eliminación de duplicados). Aún no está modelado para análisis.

## Capa de Integración / Core (Gold)

Datos modelados dimensionalmente (hechos y dimensiones) o en 3FN, listos para el negocio. Es la capa que consultan las herramientas de BI.

## Capa de Data Marts

Subconjuntos de la capa Gold orientados a áreas específicas (ventas, marketing). Pueden ser vistas o tablas físicas.

::: center
:::

# Metadatos y Gobernanza

Los metadatos son datos sobre los datos: origen, transformaciones, significados, calidad. Son esenciales para la gobernanza.

## Tipos de metadatos

- **Técnicos**: esquemas, tipos de datos, particiones, estadísticas.

- **De negocio**: definiciones, reglas de cálculo, propietarios.

- **Operacionales**: logs de ejecución, tiempos de carga.

## Herramientas de catálogo

- Apache Atlas, Amundsen, DataHub.

- Catálogos nativos de nube (AWS Glue, Google Data Catalog).

## Linaje de datos

El linaje (data lineage) muestra el flujo de los datos desde su origen hasta su consumo. Es clave para auditoría y solución de problemas. Se puede implementar con herramientas especializadas o mediante metadatos.

# Integración con Herramientas de Visualización

Los datos del data warehouse se consumen a través de herramientas de Business Intelligence (BI).

## Herramientas populares

- Power BI, Tableau, Looker, Qlik.

- Conexiones nativas a los principales DW (BigQuery, Redshift, Snowflake).

- Uso de conectores ODBC/JDBC.

## Rendimiento y costos

- Las consultas de BI pueden ser costosas si no están optimizadas.

- Se recomienda usar agregaciones precalculadas, vistas materializadas.

- Algunos DW ofrecen caché de resultados (BigQuery, Snowflake).

## Ejemplo: Power BI conectado a Snowflake

1.  Configurar el conector de Snowflake en Power BI.

2.  Importar o usar DirectQuery.

3.  Crear dashboard con medidas DAX.

Costos aproximados: Snowflake cobra por almacenamiento y cómputo (por segundo). Power BI tiene licencias por usuario.

# Dimensionamiento de Almacenamiento y Servidores

Estimar el tamaño de un data warehouse es crucial para presupuestar y diseñar.

## Factores a considerar

- Volumen diario de datos.

- Retención histórica.

- Compresión (Parquet, columnar).

- Índices y particionamiento.

- Replicación y backups.

### Ejemplo de cálculo

Una empresa genera 100 GB de datos al día. Se retienen 5 años.

- Datos diarios: 100 GB.

- Datos anuales: 100 GB \* 365 = 36.5 TB.

- 5 años: 182.5 TB.

- Con compresión columnar (factor 5:1): 3̃6.5 TB.

- Más espacio para índices, staging, etc.: 50 TB.

## Costos aproximados en la nube

- Almacenamiento en S3:  \$0.023/GB-mes. 50 TB = \$1,150/mes.

- Computo en Redshift: desde \$0.25/hora por nodo.

- Snowflake: almacenamiento similar, cómputo por segundo (créditos).

Estos costos son orientativos y varían por región y tipo de instancia.

# Arquitecturas Híbridas: SQL y NoSQL

En proyectos reales, conviven múltiples motores. Cada uno se usa para lo que mejor sabe hacer.

## Casos de uso combinados

- **PostgreSQL (OLTP)** + **MongoDB** (catálogo de productos flexible) + **Elasticsearch** (búsqueda) + **Redshift** (análisis).

- **Cassandra** (escritura masiva) + **Spark** (procesamiento) + **PostgreSQL** (reportes).

## Sincronización entre motores

- CDC desde la base transaccional a Kafka, luego a Elasticsearch y al DW.

- Herramientas como Debezium, Kafka Connect.

- Batch periódicos con scripts.

## Ejemplo: E-commerce

1.  Pedidos en PostgreSQL.

2.  Productos y reviews en MongoDB.

3.  Búsqueda con Elasticsearch.

4.  Análisis de ventas en Snowflake (carga diaria desde PostgreSQL).

# Pipelines, Jobs y Orquestación

Un pipeline de datos es una serie de pasos que transforman y mueven datos. Los pipelines se ejecutan mediante jobs (tareas) y son orquestados por herramientas.

## Definiciones

- **Pipeline**: Flujo completo de datos (extracción, transformación, carga).

- **Job**: Unidad de trabajo dentro del pipeline (por ejemplo, una tarea de Spark).

- **Task**: Paso individual (ejecutar un script, una consulta SQL).

- **DAG**: Directed Acyclic Graph, la representación de dependencias entre tareas.

## Orquestadores populares

- **Apache Airflow**: El más usado. Define DAGs en Python.

- **Prefect**, **Dagster**: Alternativas modernas.

- **AWS Step Functions**, **Google Cloud Composer** (Airflow gestionado).

- **Apache Oozie** (Hadoop, en desuso).

### Ejemplo de DAG en Airflow

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime

def extraer():
    # código para extraer datos
    pass

def transformar():
    # código para transformar
    pass

def cargar():
    # código para cargar
    pass

with DAG('etl_diario', start_date=datetime(2025,1,1), schedule_interval='@daily') as dag:
    t1 = PythonOperator(task_id='extraer', python_callable=extraer)
    t2 = PythonOperator(task_id='transformar', python_callable=transformar)
    t3 = PythonOperator(task_id='cargar', python_callable=cargar)
    t1 >> t2 >> t3
```

## Dónde se ejecutan los pipelines

- En servidores virtuales (EC2, VMs).

- En clústeres gestionados (EMR, Databricks).

- En servicios serverless (AWS Glue, Google Dataflow).

# Ejercicios Resueltos

## Ejercicio 1: Diseño de capas para un DW de ventas

**Enunciado:** Una tienda online quiere construir un data warehouse. Proponga un diseño de capas (raw, staging, core) y las transformaciones típicas en cada una.

**Solución:**

- **Raw**: Se almacenan los archivos CSV/JSON de ventas, productos, clientes tal como llegan del sistema transaccional. Particionado por fecha.

- **Staging**: Se leen los raw, se asignan tipos de datos correctos, se eliminan duplicados, se unifican formatos de fechas. Se guardan en tablas temporales o en Parquet.

- **Core**: Se construyen tablas de hechos (ventas) y dimensiones (cliente, producto, tiempo) aplicando modelado dimensional. Se aplican SCD tipo 2 para dimensiones cambiantes.

## Ejercicio 2: Implementar carga incremental con SQL

**Enunciado:** Escribir una consulta SQL que actualice la tabla `dw_ventas` con los datos de `stg_ventas` del día, usando MERGE.

**Solución:**

```sql
MERGE INTO dw_ventas AS target
USING stg_ventas AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET target.monto = source.monto, target.fecha_actualizacion = CURRENT_TIMESTAMP
WHEN NOT MATCHED THEN
    INSERT (id, fecha, monto, fecha_actualizacion)
    VALUES (source.id, source.fecha, source.monto, CURRENT_TIMESTAMP);
```

## Ejercicio 3: Estimar costo de almacenamiento

**Enunciado:** Una empresa genera 500 GB de datos al día. Retiene 3 años. Estimar el almacenamiento necesario con compresión 4:1 y costo mensual en S3 (a \$0.023/GB).

**Solución:**

- Datos diarios: 500 GB.

- Anual: 500 GB \* 365 = 182.5 TB.

- 3 años: 547.5 TB.

- Con compresión 4:1: 136.9 TB.

- Espacio adicional (10%): 150.6 TB.

- Costo mensual: 150.6 \* 1024 GB/TB \* 0.023 = 150.6 \* 23.552 ≈ \$3,546/mes.

## Ejercicio 4: Definir un pipeline con Airflow

**Enunciado:** Crear un DAG de Airflow con tres tareas: extraer de API, transformar con Spark, cargar en Redshift.

**Solución:**

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.providers.amazon.aws.operators.redshift import RedshiftSQLOperator
from airflow.operators.python import PythonOperator
from datetime import datetime
import requests

def extraer_api():
    response = requests.get('https://api.ejemplo.com/data')
    with open('/tmp/data.json', 'w') as f:
        f.write(response.text)

with DAG('pipeline_ventas', start_date=datetime(2025,1,1), schedule='@daily') as dag:
    extraer = PythonOperator(task_id='extraer', python_callable=extraer_api)
    transformar = SparkSubmitOperator(
        task_id='transformar',
        application='/path/to/spark_job.py',
        conn_id='spark_default'
    )
    cargar = RedshiftSQLOperator(
        task_id='cargar',
        sql='CALL cargar_ventas();',
        redshift_conn_id='redshift_default'
    )
    extraer >> transformar >> cargar
```
