
## Sesión 14
# GESTIÓN DE ERRORES, RECUPERACIÓN Y ARQUITECTURAS HÍBRIDAS


**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción

En esta última sesión del curso, integraremos todos los conocimientos adquiridos para abordar los desafíos reales de la gestión de bases de datos en producción. Nos centraremos en cómo prevenir y recuperarse de errores catastróficos, cómo diseñar arquitecturas híbridas que combinen lo mejor de SQL, NoSQL y bases de datos vectoriales, y cómo desplegar estos sistemas de manera confiable y escalable. Este capítulo final es una guía para convertirte en un profesional capaz de enfrentar incidentes, diseñar soliones resilientes y mantener la consistencia en entornos políglotas.

## Objetivos de la sesión

- Conocer las operaciones de alto riesgo y las mejores prácticas para evitarlas.

- Entender los mecanismos de recuperación ante desastres: backups, snapshots, point-in-time recovery.

- Analizar los desafíos específicos de la nube: latencia, costos de egreso, data drift.

- Diseñar arquitecturas híbridas que integren motores SQL, NoSQL y vectoriales.

- Aprender a orquestar múltiples bases de datos con contenedores e infraestructura como código.

- Implementar sincronización entre sistemas mediante CDC y triggers.

- Resolver ejercicios prácticos que integran todos los conceptos.

# Operaciones de Alto Riesgo y Prevención

En entornos productivos, ciertos comandos pueden causar daños irreparables si se ejecutan sin control. Conocerlos y establecer barreras es fundamental.

## DROP vs. TRUNCATE

- `DROP TABLE`: Elimina la tabla y su estructura (metadata). Los datos no se pueden recuperar fácilmente (excepto desde backups).

- `TRUNCATE TABLE`: Elimina todas las filas de una tabla de forma rápida, pero conserva la estructura. No genera registros de cada fila eliminada, por lo que no se puede deshacer con `ROLLBACK` en la mayoría de los motores.

::: center
  **Operación**      **¿Elimina estructura?**   **¿Registra por fila?**
  ------------------ -------------------------- -----------------------------
  DROP               Sí                         No (metadata)
  TRUNCATE           No                         No (solo desasigna páginas)
  DELETE sin WHERE   No                         Sí (puede ser lento)
:::

### Buenas prácticas

- Siempre realizar un `SELECT` antes de un `DELETE` o `UPDATE` para verificar la condición.

- Usar transacciones: `BEGIN; DELETE ...; ROLLBACK;` para probar.

- En entornos críticos, deshabilitar permisos de `DROP` y `TRUNCATE` para usuarios regulares.

- Implementar una política de \"soft delete\" (marcar filas como eliminadas en lugar de borrarlas físicamente).

## UPDATE/DELETE sin WHERE

El error humano más común. Por ejemplo:

```sql
-- Peligro: actualiza todos los salarios
UPDATE empleados SET salario = 0;

-- Correcto: con WHERE
UPDATE empleados SET salario = 0 WHERE id = 123;
```

### Estrategias de mitigación

- Usar `BEGIN` y `ROLLBACK` en entornos de prueba.

- Configurar `sql_safe_updates` en MySQL (requiere WHERE con key).

- En PostgreSQL, se puede usar `row_security` o triggers que impidan actualizaciones masivas.

- Auditoría de consultas: herramientas como pgaudit registran comandos peligrosos.

## Prevención mediante scripts de validación

Antes de ejecutar un script en producción, se debe:

1.  Hacer una copia de seguridad.

2.  Ejecutar el script en un entorno de staging idéntico.

3.  Usar herramientas de CI/CD con aprobación manual.

4.  Incluir verificaciones de integridad (ej. contar filas afectadas).

# Mecanismos de Resiliencia y Recuperación ante Desastres (DRP)

Un plan de recuperación ante desastres (DRP) garantiza que los datos puedan restaurarse después de un fallo catastrófico.

## Rollback y Transacciones

Las transacciones permiten deshacer cambios si algo sale mal.

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
-- Si todo va bien:
COMMIT;
-- Si ocurre un error:
ROLLBACK;
```

Es crucial que las aplicaciones manejen correctamente las transacciones.

## Backups y Snapshots

### Backups lógicos

Consisten en volcados de datos en formato SQL o de texto. Ejemplo con `pg_dump`:

```bash
pg_dump -U usuario -d mi_db > backup.sql
```

Ventajas: portabilidad, se puede restaurar en versiones diferentes. Desventajas: lentos para bases grandes.

### Backups físicos (snapshots)

Capturas instantáneas del sistema de archivos o del almacenamiento. Ejemplo: snapshot de EBS en AWS. Son rápidos y consistentes, pero dependen del proveedor.

### Backups incrementales

Solo respaldan los cambios desde el último backup completo. Herramientas como pg_basebackup (PostgreSQL) o binlog (MySQL) permiten esto.

## Point-in-Time Recovery (PITR)

Permite restaurar la base a un momento exacto antes de un incidente. Requiere:

- Un backup completo.

- Los archivos de log de transacciones (WAL en PostgreSQL, binlog en MySQL) desde ese backup hasta el momento deseado.

Ejemplo en PostgreSQL:

1.  Configurar `wal_level = replica` y archiving.

2.  Restaurar el backup base.

3.  Aplicar WAL hasta el momento deseado con `recovery_target_time`.

## Ejemplo práctico: Restauración PITR en PostgreSQL

```bash
# 1. Backup base
pg_basebackup -D /backup/base -F t -z -P

# 2. Configurar recovery.conf (en versiones antiguas) o postgresql.auto.conf
# en postgresql.auto.conf:
restore_command = 'cp /wal_archive/%f %p'
recovery_target_time = '2025-03-20 14:30:00'

# 3. Iniciar PostgreSQL, que aplicará los WAL hasta ese momento.
```

# Desafíos en la Nube y Data Drift

## Latencia y Egreso

En la nube, mover grandes volúmenes de datos entre regiones o fuera del proveedor tiene costos significativos (egress). Estrategias:

- Mantener los datos cerca de donde se procesan (misma región).

- Usar transferencias internas (por ejemplo, de S3 a EC2 sin costo).

- Comprimir datos antes de transferir.

Ejemplo: mover 10 TB desde AWS a otra nube puede costar cientos de dólares.

## Data Drift (Deriva de Datos)

En modelos de IA, el data drift ocurre cuando la distribución de los datos cambia con el tiempo, degradando el rendimiento. Es un \"error\" silencioso. Para detectarlo:

- Monitorear estadísticas de los datos (media, desviación, etc.).

- Comparar distribuciones de entrenamiento vs. producción (test de Kolmogorov-Smirnov, etc.).

- Herramientas: Evidently, Great Expectations, WhyLabs.

La gobernanza activa y el reentrenamiento periódico son clave.

# Arquitecturas Híbridas: Integración de SQL, NoSQL y Vectorial

En proyectos de IA a gran escala, un solo motor no es suficiente. Se despliega un ecosistema políglota.

## Capa Transaccional (SQL)

Motor principal para datos maestros, ACID, consistencia fuerte. Ejemplos: PostgreSQL, Oracle, SQL Server.

## Capa de Velocidad (NoSQL)

Bases de datos en memoria o clave-valor para sesiones, caché, baja latencia. Ejemplos: Redis, Memcached.

## Capa de Memoria Semántica (Vectorial)

Almacena embeddings para búsqueda por similitud. Ejemplos: Milvus, Pinecone, pgvector.

::: center
:::

## Orquestación de Persistencia Políglota

### Contenedores (Docker)

Cada motor se ejecuta en su contenedor, con volúmenes persistentes. Ejemplo docker-compose.yml:

```bash
version: '3'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: admin
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7
  milvus:
    image: milvusdb/milvus:latest
    ...
volumes:
  pgdata:
```

### Kubernetes

Para orquestación a gran escala, se usan StatefulSets y Services.

## Infraestructura como Código (IaC)

Herramientas como Terraform o CloudFormation definen la infraestructura de forma declarativa.

```bash
# Ejemplo Terraform para AWS RDS
resource "aws_db_instance" "postgres" {
  allocated_storage = 20
  engine            = "postgres"
  instance_class     = "db.t3.micro"
  username          = "admin"
  password          = "password"
  skip_final_snapshot = true
}
```

Ventajas: replicable, versionable, evita errores manuales.

## Aislamiento de Redes

Usar VPCs, subredes privadas, grupos de seguridad para que los motores se comuniquen internamente sin exposición pública.

## Gestión de Volúmenes Cruzados

En entornos orquestados, los volúmenes deben ser compartidos o tener backups consistentes. Por ejemplo, usar PersistentVolumeClaims en Kubernetes.

# Sincronización y Consistencia entre Motores

El mayor desafío es mantener la coherencia entre la base transaccional y la vectorial.

## Disparadores (Triggers) y CDC

### Triggers en SQL

Un trigger puede insertar un registro en una tabla de cambios, que luego un proceso externo (ej. Python) lee y actualiza la base vectorial.

```sql
CREATE TABLE cambios (
    id SERIAL PRIMARY KEY,
    tabla TEXT,
    operacion CHAR(1),
    data JSONB,
    procesado BOOLEAN DEFAULT FALSE
);

CREATE OR REPLACE FUNCTION log_cambio() RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO cambios (tabla, operacion, data) VALUES (TG_TABLE_NAME, TG_OP, row_to_json(NEW));
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_productos AFTER INSERT OR UPDATE OR DELETE ON productos
FOR EACH ROW EXECUTE FUNCTION log_cambio();
```

Luego, un worker consulta `cambios` y actualiza la base vectorial.

### Change Data Capture (CDC)

Herramientas como Debezium capturan cambios directamente del log de transacciones (WAL, binlog) y los publican en Kafka. Luego, un consumidor (Kafka Connect) actualiza la base vectorial.

::: center
:::

## Consistencia Eventual

La replicación asíncrona introduce latencia: los cambios pueden tardar segundos en reflejarse en la base vectorial. Las aplicaciones deben tolerar esta eventual consistencia. Estrategias:

- Marcar datos como \"no indexados\" hasta que se confirme la sincronización.

- Usar un TTL en la caché.

- Aceptar que las búsquedas pueden no incluir los datos más recientes (trade-off).

# Ejercicios Resueltos

## Ejercicio 1: Prevención de DROP accidental

**Enunciado:** En PostgreSQL, crear un trigger que impida ejecutar DROP TABLE en una tabla crítica.

**Solución:** No se puede evitar DROP directamente con un trigger (se ejecuta a nivel de DDL). En su lugar, se pueden revocar permisos:

```sql
REVOKE DROP ON TABLE productos FROM usuario;
```

O usar event triggers (PostgreSQL 9.3+) para capturar DDL:

```sql
CREATE OR REPLACE FUNCTION abort_drop() RETURNS event_trigger AS $$
BEGIN
  RAISE EXCEPTION 'No se permite DROP en esta base de datos';
END;
$$ LANGUAGE plpgsql;

CREATE EVENT TRIGGER abort_drop_trigger ON sql_drop
  EXECUTE FUNCTION abort_drop();
```

(Nota: esto impide cualquier DROP, puede ser demasiado restrictivo.)

## Ejercicio 2: Configurar Point-in-Time Recovery en PostgreSQL con Docker

**Enunciado:** Levantar un contenedor PostgreSQL, configurar archiving WAL, simular un error y restaurar a un momento antes.

**Solución:**

```bash
# 1. Crear red y volumen
docker network create pitr-net
docker volume create pgdata
docker volume create walarchive

# 2. Ejecutar PostgreSQL con configuración de archiving
docker run -d --name pg-pitr \
  --network pitr-net \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_DB=test \
  -v pgdata:/var/lib/postgresql/data \
  -v walarchive:/wal_archive \
  postgres:15 \
  -c wal_level=replica \
  -c archive_mode=on \
  -c archive_command='cp %p /wal_archive/%f' \
  -c archive_timeout=60

# 3. Crear datos y luego un punto de tiempo
docker exec -it pg-pitr psql -U postgres -d test -c "CREATE TABLE datos (id INT); INSERT INTO datos VALUES (1); SELECT pg_switch_wal();"
# Esperar un minuto
docker exec -it pg-pitr psql -U postgres -d test -c "INSERT INTO datos VALUES (2); SELECT pg_switch_wal();"

# 4. Simular un desastre (borrar la tabla)
docker exec -it pg-pitr psql -U postgres -d test -c "DROP TABLE datos;"

# 5. Detener el contenedor y restaurar
docker stop pg-pitr
# Crear un nuevo contenedor para restaurar
docker run -d --name pg-restore \
  --network pitr-net \
  -v pgdata:/var/lib/postgresql/data \
  -v walarchive:/wal_archive \
  postgres:15 \
  echo "restore"

# En realidad, para restaurar se necesita un backup base y luego recovery.
# Simplificamos: suponiendo que tenemos un backup base, copiamos los archivos.
```

Nota: Un ejemplo completo sería muy extenso; se sugiere usar herramientas como pg_basebackup.

## Ejercicio 3: Integración de PostgreSQL con Redis mediante CDC simulado

**Enunciado:** Simular que al insertar un producto en PostgreSQL, se actualiza una clave en Redis con el nombre del producto. Usar un trigger y un script Python.

**Solución:**

```sql
-- En PostgreSQL, tabla productos
CREATE TABLE productos (id SERIAL PRIMARY KEY, nombre TEXT);
-- Tabla de cambios
CREATE TABLE cambios_productos (
    id SERIAL PRIMARY KEY,
    producto_id INT,
    nombre TEXT,
    operacion CHAR(1),
    procesado BOOLEAN DEFAULT FALSE
);

CREATE OR REPLACE FUNCTION log_producto() RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO cambios_productos (producto_id, nombre, operacion) VALUES (NEW.id, NEW.nombre, 'I');
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO cambios_productos (producto_id, nombre, operacion) VALUES (NEW.id, NEW.nombre, 'U');
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO cambios_productos (producto_id, nombre, operacion) VALUES (OLD.id, OLD.nombre, 'D');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trig_productos AFTER INSERT OR UPDATE OR DELETE ON productos
FOR EACH ROW EXECUTE FUNCTION log_producto();
```

Script Python:

```python
import psycopg2
import redis
import time

conn_pg = psycopg2.connect(dbname="test", user="postgres")
cur_pg = conn_pg.cursor()
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

while True:
    cur_pg.execute("SELECT id, producto_id, nombre, operacion FROM cambios_productos WHERE procesado = FALSE LIMIT 1")
    row = cur_pg.fetchone()
    if row:
        cambio_id, prod_id, nombre, op = row
        if op in ('I', 'U'):
            r.set(f"producto:{prod_id}", nombre)
        elif op == 'D':
            r.delete(f"producto:{prod_id}")
        cur_pg.execute("UPDATE cambios_productos SET procesado = TRUE WHERE id = %s", (cambio_id,))
        conn_pg.commit()
        print(f"Sincronizado {prod_id}")
    time.sleep(1)
```

## Ejercicio 4: Despliegue de arquitectura híbrida con Docker Compose

**Enunciado:** Crear un docker-compose.yml que levante PostgreSQL, Redis y una aplicación Python que los use.

**Solución:**

```bash
version: '3'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: test
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-net

  redis:
    image: redis:7
    networks:
      - app-net

  app:
    build: ./app
    depends_on:
      - postgres
      - redis
    environment:
      POSTGRES_HOST: postgres
      REDIS_HOST: redis
    networks:
      - app-net

networks:
  app-net:
    driver: bridge

volumes:
  pgdata:
```

Con un `Dockerfile` para la app Python que ejecute el script de sincronización.

## Ejercicio 5: Detección de data drift con Python

**Enunciado:** Comparar la distribución de una variable en un conjunto de entrenamiento y en producción usando el test de Kolmogorov-Smirnov.

**Solución:**

```python
import numpy as np
from scipy.stats import ks_2samp

# Datos de entrenamiento (ejemplo)
train = np.random.normal(100, 15, 1000)
# Datos de producción reciente
prod = np.random.normal(105, 18, 500)

stat, p_value = ks_2samp(train, prod)
alpha = 0.05
if p_value < alpha:
    print("Se detecta data drift (p=%f)" % p_value)
else:
    print("No hay evidencia de drift")
```
