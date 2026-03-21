
## Sesión 13
# DESPLIEGUE Y ALTA DISPONIBILIDAD
## Contenedores, orquestación, replicación, migración y estrategias HA

**Autor:** Carlos César Sánchez Coronel  
**Fecha:** 2026

---

# Introducción

En las sesiones anteriores hemos cubierto desde los fundamentos del modelado hasta las bases de datos vectoriales. Ahora nos enfocamos en cómo llevar estos sistemas a producción: cómo desplegarlos de manera confiable, escalable y con alta disponibilidad. Esta sesión aborda el uso de contenedores (Docker) para estandarizar entornos, orquestación con Kubernetes para gestionar clústeres, estrategias de alta disponibilidad (réplicas, failover, clustering), y la migración de bases de datos entre sistemas y entornos (on-premise a nube). También veremos herramientas específicas de cada motor para replicación y alta disponibilidad, así como balanceadores de carga. Todo ello con ejemplos prácticos y ejercicios resueltos.

## Objetivos de la sesión

- Comprender la importancia de la contenedorización para bases de datos.

- Aprender a desplegar SGBD en Docker y configurar persistencia.

- Conocer los conceptos básicos de Kubernetes aplicados a bases de datos.

- Diferenciar estrategias de alta disponibilidad: réplicas de lectura, failover, clustering.

- Configurar replicación en motores populares (PostgreSQL, MySQL).

- Entender el balanceo de carga para bases de datos.

- Analizar opciones en la nube (RDS, Cloud SQL) y comparar con on-premise.

- Planificar y ejecutar migraciones de bases de datos entre distintos motores o entornos.

- Realizar ejercicios prácticos paso a paso.

# Contenedores y Bases de Datos

## ¿Por qué contenedores?

Los contenedores (Docker) permiten empaquetar una aplicación con todas sus dependencias, garantizando que se ejecute de la misma manera en cualquier entorno. Para bases de datos, los contenedores facilitan:

- Entornos de desarrollo y prueba idénticos a producción.

- Rápido aprovisionamiento y desmontaje.

- Aislamiento de recursos.

- Integración con pipelines CI/CD.

## Dockerizando una Base de Datos

### Ejemplo: PostgreSQL en Docker

```bash
# Descargar imagen oficial
docker pull postgres:15

# Ejecutar contenedor con variables de entorno y volumen persistente
docker run -d \
  --name mi-postgres \
  -e POSTGRES_PASSWORD=mi_password \
  -e POSTGRES_USER=usuario \
  -e POSTGRES_DB=mi_db \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15
```

Explicación:

- `-d`: modo detached (fondo).

- `--name`: nombre del contenedor.

- `-e`: variables de entorno para configurar usuario, contraseña y base de datos inicial.

- `-v pgdata:/var/lib/postgresql/data`: volumen persistente (en este caso, un volumen nombrado) para que los datos sobrevivan al contenedor.

- `-p 5432:5432`: mapea el puerto del contenedor al host.

### Volúmenes en Docker

Es crucial usar volúmenes para persistir datos. Tipos:

- **Volúmenes nombrados**: gestionados por Docker, ubicados en `/var/lib/docker/volumes/`. Ejemplo: `-v pgdata:/var/lib/postgresql/data`.

- **Bind mounts**: mapean un directorio del host. Ejemplo: `-v /ruta/en/host:/var/lib/postgresql/data`.

Recomendación: usar volúmenes nombrados para producción, ya que Docker los gestiona mejor.

### Redes en Docker

Para que múltiples contenedores se comuniquen, se crean redes personalizadas.

```bash
# Crear red
docker network create mi-red

# Ejecutar contenedor en esa red
docker run -d --name postgres --network mi-red -e POSTGRES_PASSWORD=... postgres

# Otro contenedor (ej. aplicación) en la misma red puede acceder usando el nombre del contenedor como hostname
```

### Consideraciones de Rendimiento

Ejecutar bases de datos en contenedores tiene overhead mínimo si se configuran correctamente. Sin embargo, en producción se recomienda:

- Usar volúmenes rápidos (SSD).

- Limitar recursos con flags como `--memory` y `--cpus`.

- Ajustar parámetros del kernel según necesidades (ej. `shared_buffers` en PostgreSQL).

## Orquestación con Kubernetes

Kubernetes (K8s) es el estándar para orquestar contenedores. Para bases de datos, Kubernetes ofrece:

- Despliegues declarativos (Deployments, StatefulSets).

- Auto-escalado.

- Balanceo de carga interno (Services).

- Auto-reparación (reinicio de contenedores fallidos).

### StatefulSets para Bases de Datos

A diferencia de Deployments, StatefulSet garantiza identidades estables (nombres de host persistentes) y orden en el despliegue/escalado. Es la opción recomendada para bases de datos.

### Ejemplo: PostgreSQL en Kubernetes

Archivo YAML para un StatefulSet:

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          value: "mi_password"
        - name: POSTGRES_USER
          value: "usuario"
        - name: POSTGRES_DB
          value: "mi_db"
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: pgdata
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

Este YAML define un StatefulSet que crea un PersistentVolumeClaim por cada réplica, asegurando almacenamiento persistente. Luego se necesita un Service para acceder:

```bash
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

### Operadores para Bases de Datos en Kubernetes

Para gestionar bases de datos de manera más avanzada (backups, replicación automática), existen operadores como:

- **PostgreSQL Operator** (Crunchy Data, Zalando).

- **MySQL Operator** (Oracle).

- **Cassandra Operator** (DataStax).

Estos operadores extienden Kubernetes para manejar tareas complejas.

# Estrategias de Alta Disponibilidad (HA)

La alta disponibilidad busca minimizar el tiempo de inactividad de un sistema. En bases de datos, las estrategias comunes son:

## Réplicas de Lectura

Consisten en tener una o más copias (réplicas) de la base de datos principal que reciben los cambios en tiempo real (o casi). Las consultas de solo lectura pueden dirigirse a las réplicas, descargando al nodo principal.

### Beneficios

- Escalabilidad horizontal de lecturas.

- Aislamiento de cargas: consultas analíticas pesadas no afectan al transaccional.

- Mayor disponibilidad: si el principal falla, una réplica puede promoverse.

### Implementaciones comunes

- **PostgreSQL**: Replicación física (streaming replication) o lógica.

- **MySQL**: Replicación asíncrona (binlog) o semisíncrona.

- **MongoDB**: Réplicas (replica sets).

## Failover

El failover es el proceso de conmutar automáticamente a un nodo secundario cuando el principal falla. Puede ser:

- **Manual**: Un administrador promueve la réplica.

- **Automático**: Un sistema de monitoreo detecta la falla y ejecuta la promoción (ej. Patroni, Orchestrator, MHA).

### Consideraciones

- Tiempo de detección y promoción (RTO).

- Posible pérdida de datos (RPO) si la replicación es asíncrona.

- Split-brain: evitar que dos nodos se consideren principales simultáneamente.

## Clustering

Un cluster de bases de datos puede ser:

- **Activo-Pasivo**: Un nodo activo, otro(s) pasivo(s) que toman el control si falla el activo. Ejemplo: PostgreSQL con Patroni + etcd.

- **Activo-Activo**: Todos los nodos aceptan escrituras, con mecanismos de resolución de conflictos. Ejemplo: MySQL Cluster (NDB), Oracle RAC.

## Balanceo de Carga

Para distribuir el tráfico entre réplicas o nodos, se utilizan balanceadores:

- **ProxySQL**: balanceo para MySQL, con enrutamiento basado en consultas.

- **HAProxy**: balanceo TCP genérico, puede usarse para PostgreSQL o MySQL.

- **pgpool-II**: balanceo y pooling para PostgreSQL.

# Replicación en Motores Específicos

## PostgreSQL: Streaming Replication

Configuración básica (asíncrona):

1.  En el primario, crear usuario de replicación:

    ``` {.sql language="SQL"}
    CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'password';
    ```

2.  Editar `postgresql.conf`:

    ``` {.Bash language="Bash"}
    wal_level = replica
    max_wal_senders = 5
    wal_keep_size = 1GB
    ```

3.  En `pg_hba.conf` permitir conexiones de replicación:

    ``` {.Bash language="Bash"}
    host replication replicator 192.168.1.0/24 md5
    ```

4.  Reiniciar PostgreSQL.

5.  En el secundario, realizar un backup base:

    ``` {.Bash language="Bash"}
    pg_basebackup -h primario -D /var/lib/postgresql/data -U replicator -v -P --wal-method=stream
    ```

6.  Crear archivo `standby.signal` y configurar `postgresql.auto.conf` con la conexión al primario:

    ``` {.Bash language="Bash"}
    primary_conninfo = 'host=primario port=5432 user=replicator password=password'
    ```

7.  Iniciar el secundario.

Para failover automático se puede usar Patroni o repmgr.

## MySQL: Group Replication

Group Replication proporciona replicación multi-maestro con detección automática de fallos. Ejemplo de configuración (InnoDB Cluster):

1.  Configurar cada nodo con un server_id único y habilitar GTID.

2.  Instalar plugin:

    ``` {.sql language="SQL"}
    INSTALL PLUGIN group_replication SONAME 'group_replication.so';
    ```

3.  Configurar variables (seed, credenciales, etc.).

4.  Iniciar Group Replication en cada nodo.

Para simplificar, se puede usar MySQL InnoDB Cluster con MySQL Shell.

## MongoDB: Replica Sets

Un replica set es un grupo de servidores mongod con un primario y varios secundarios. Configuración básica:

1.  Iniciar cada mongod con la opción `--replSet "miSet"`.

2.  Conectarse a uno y ejecutar:

    ``` {.JavaScript language="JavaScript"}
    rs.initiate()
    rs.add("host2:27017")
    rs.add("host3:27017")
    ```

El failover es automático: si el primario cae, los secundarios eligen uno nuevo.

## Oracle: Data Guard

Oracle Data Guard mantiene una base de datos standby (física o lógica). Puede ser síncrona (protección máxima) o asíncrona. La conmutación puede ser manual o automática con Fast-Start Failover.

## SQL Server: Always On Availability Groups

Always On permite grupos de disponibilidad con réplicas síncronas o asíncronas, con failover automático si se usa un cluster de Windows.

# Balanceo de Carga Práctico

### Ejemplo con HAProxy para PostgreSQL

Configuración de HAProxy (archivo `/etc/haproxy/haproxy.cfg`):

```bash
frontend pgsql
    bind *:5432
    default_backend pgsql_backend

backend pgsql_backend
    balance roundrobin
    option pgsql-check user haproxy
    server pg1 192.168.1.10:5432 check
    server pg2 192.168.1.11:5432 check
    server pg3 192.168.1.12:5432 check
```

Se necesita un usuario `haproxy` en PostgreSQL para las comprobaciones de salud.

### ProxySQL para MySQL

ProxySQL permite enrutamiento basado en consultas y pooling de conexiones. Configuración básica:

```sql
INSERT INTO mysql_servers (hostgroup_id, hostname, port) VALUES (10, '192.168.1.10', 3306);
INSERT INTO mysql_servers (hostgroup_id, hostname, port) VALUES (10, '192.168.1.11', 3306);
INSERT INTO mysql_users (username, password, default_hostgroup) VALUES ('app', 'pass', 10);
LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
```

# Despliegue en la Nube vs On-Premise

## Servicios Gestionados (PaaS)

- **AWS RDS**: Soporta múltiples motores, replicación, backups automáticos, failover multi-AZ.

- **Google Cloud SQL**: PostgreSQL, MySQL, SQL Server con alta disponibilidad.

- **Azure SQL Database**: Base de datos como servicio, escalable.

Ventajas: menor overhead administrativo, escalado sencillo, alta disponibilidad integrada. Desventajas: menos control, posible vendor lock-in, costos.

## On-Premise

La empresa gestiona su propio hardware y software. Ventajas: control total, cumplimiento de normativas estrictas. Desventajas: inversión inicial, mantenimiento.

## Modelo Híbrido

Combinar recursos on-premise con nube, por ejemplo, para disaster recovery (DR) o picos de carga.

## Migración a la Nube

Pasos típicos:

1.  Evaluación de la base de datos (tamaño, complejidad, dependencias).

2.  Elección del servicio objetivo (RDS, Cloud SQL, etc.).

3.  Migración de datos: herramientas como AWS DMS (Database Migration Service), mysqldump + import, pg_dump + pg_restore.

4.  Pruebas de rendimiento y funcionalidad.

5.  Corte de producción.

### Ejemplo: Migrar PostgreSQL local a AWS RDS

```bash
# Dump de la base local
pg_dump -U usuario -h localhost mi_db > mi_db.sql

# Restaurar en RDS (previamente crear instancia)
psql -U usuario -h mi-instance.rds.amazonaws.com -d mi_db < mi_db.sql
```

Para migraciones sin downtime, se puede usar AWS DMS con replicación continua.

# Ejercicios Resueltos

## Ejercicio 1: Desplegar PostgreSQL con Docker y configurar persistencia

**Enunciado:** Crear un contenedor PostgreSQL con un volumen persistente, crear una base de datos y una tabla, insertar datos, detener el contenedor, volver a iniciarlo y comprobar que los datos persisten.

**Solución:**

```bash
# 1. Crear volumen
docker volume create pgdata

# 2. Ejecutar contenedor
docker run -d --name postgres1 \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=testdb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# 3. Conectar y crear tabla
docker exec -it postgres1 psql -U admin -d testdb
# Dentro de psql:
CREATE TABLE personas (id SERIAL, nombre TEXT);
INSERT INTO personas (nombre) VALUES ('Ana'), ('Luis');
SELECT * FROM personas;
\q

# 4. Detener y eliminar contenedor
docker stop postgres1
docker rm postgres1

# 5. Crear nuevo contenedor con el mismo volumen
docker run -d --name postgres2 \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=testdb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# 6. Verificar datos
docker exec -it postgres2 psql -U admin -d testdb -c "SELECT * FROM personas;"
# Deberían aparecer Ana y Luis
```

## Ejercicio 2: Configurar replicación PostgreSQL con dos contenedores Docker

**Enunciado:** En una misma máquina, levantar dos contenedores PostgreSQL (primario y réplica) y configurar streaming replication asíncrona.

**Solución:**

```bash
# Red común
docker network create pg-net

# Primario
docker run -d --name pg-primary \
  --network pg-net \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=testdb \
  -v pg-primary-data:/var/lib/postgresql/data \
  postgres:15

# Configurar primario
docker exec -it pg-primary bash
# Editar postgresql.conf (usar echo o sed)
echo "wal_level = replica" >> /var/lib/postgresql/data/postgresql.conf
echo "max_wal_senders = 5" >> /var/lib/postgresql/data/postgresql.conf
echo "wal_keep_size = 1GB" >> /var/lib/postgresql/data/postgresql.conf
# En pg_hba.conf permitir replicación desde la red
echo "host replication all 172.16.0.0/12 md5" >> /var/lib/postgresql/data/pg_hba.conf
exit
docker restart pg-primary

# Crear usuario replicador
docker exec -it pg-primary psql -U admin -c "CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replica';"

# En la réplica, hacer basebackup
docker run -d --name pg-replica \
  --network pg-net \
  -v pg-replica-data:/var/lib/postgresql/data \
  postgres:15 echo "Esperando"
docker exec -it pg-replica bash
# Limpiar datos por defecto
rm -rf /var/lib/postgresql/data/*
# Realizar backup
pg_basebackup -h pg-primary -U replicator -D /var/lib/postgresql/data -v -P --wal-method=stream
# Crear archivo standby.signal
touch /var/lib/postgresql/data/standby.signal
# Configurar primary_conninfo
echo "primary_conninfo = 'host=pg-primary user=replicator password=replica'" >> /var/lib/postgresql/data/postgresql.auto.conf
exit
# Iniciar réplica (reiniciar contenedor)
docker stop pg-replica
docker start pg-replica

# Verificar en primario:
docker exec -it pg-primary psql -U admin -c "SELECT * FROM pg_stat_replication;"
# Debería mostrar la réplica
```

## Ejercicio 3: Failover manual en PostgreSQL con réplica

**Enunciado:** Simular la caída del primario y promover la réplica a nuevo primario.

**Solución:** Continuando del ejercicio anterior:

```bash
# Detener primario
docker stop pg-primary

# En la réplica, promover
docker exec -it pg-replica bash
# Crear archivo de trigger (si se usa trigger_file) o simplemente eliminar standby.signal
rm /var/lib/postgresql/data/standby.signal
# También se puede usar pg_ctl promote (pero dentro del contenedor no está el comando fácilmente). Alternativa: reiniciar sin standby.
# Para PostgreSQL 12+, con promote:
# En el contenedor, ejecutar:
pg_ctl promote -D /var/lib/postgresql/data
# Pero requiere que el proceso esté corriendo. Normalmente se usa:
docker exec -it pg-replica pg_ctl promote -D /var/lib/postgresql/data
# (si pg_ctl está en PATH, lo está)
# Ahora pg-replica es primario

# Verificar que acepta escrituras
docker exec -it pg-replica psql -U admin -d testdb -c "INSERT INTO personas (nombre) VALUES ('Nuevo');"
```

Para failover automático, se usarían herramientas como Patroni o repmgr.

## Ejercicio 4: Migrar base de datos MySQL local a AWS RDS

**Enunciado:** Simular la migración de una base de datos MySQL desde un contenedor local a una instancia RDS (usar MySQL local como si fuera RDS).

**Solución:**

```bash
# Levantar MySQL local (simula origen)
docker run -d --name mysql-origen \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ventas \
  -p 3306:3306 \
  mysql:8

# Crear datos de ejemplo
docker exec -it mysql-origen mysql -uroot -proot -e "USE ventas; CREATE TABLE productos (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(100)); INSERT INTO productos (nombre) VALUES ('Laptop'), ('Mouse');"

# Levantar otro MySQL local como destino (simula RDS)
docker run -d --name mysql-destino \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=ventas \
  -p 3307:3306 \
  mysql:8

# Migración con mysqldump
mysqldump -h127.0.0.1 -P3306 -uroot -proot ventas > ventas.sql
mysql -h127.0.0.1 -P3307 -uroot -proot ventas < ventas.sql

# Verificar
mysql -h127.0.0.1 -P3307 -uroot -proot -e "SELECT * FROM ventas.productos;"
```

Para una migración real a RDS, se usarían los endpoints de AWS y las credenciales correspondientes.

## Ejercicio 5: Configurar HAProxy para balancear entre dos réplicas de PostgreSQL

**Enunciado:** Tener dos contenedores PostgreSQL (réplicas) y un HAProxy que balancee las conexiones de solo lectura entre ellas.

**Solución:**

```bash
# Asumimos que ya hay dos réplicas (pg-replica1, pg-replica2) funcionando, y un primario aparte.
# Crear red
docker network create ha-net
# Conectar los contenedores a la red (si no lo están)
docker network connect ha-net pg-replica1
docker network connect ha-net pg-replica2

# Ejecutar HAProxy
docker run -d --name haproxy \
  --network ha-net \
  -p 5432:5432 \
  -v $(pwd)/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro \
  haproxy:latest

# Archivo haproxy.cfg:
cat > haproxy.cfg <<EOF
global
    log stdout format raw local0

defaults
    log global
    option tcplog
    timeout connect 5s
    timeout client 1m
    timeout server 1m

frontend pgsql
    bind *:5432
    default_backend pgsql_backend

backend pgsql_backend
    balance roundrobin
    option pgsql-check user haproxy
    server pg1 pg-replica1:5432 check
    server pg2 pg-replica2:5432 check
EOF

# En cada réplica, crear usuario haproxy
docker exec -it pg-replica1 psql -U admin -c "CREATE USER haproxy;"
docker exec -it pg-replica2 psql -U admin -c "CREATE USER haproxy;"

# Probar conexión a HAProxy
psql -h127.0.0.1 -p5432 -U admin -d testdb -c "SELECT inet_server_addr();"
# Debería alternar entre las IPs de las réplicas
```

# Glosario de Términos

Contenedor

:   Entorno aislado que ejecuta una aplicación con sus dependencias.

Docker

:   Plataforma de contenedores.

Volumen

:   Mecanismo para persistir datos en Docker.

Kubernetes

:   Orquestador de contenedores.

StatefulSet

:   Recurso de Kubernetes para aplicaciones con estado.

Replicación

:   Copia de datos de un servidor a otro.

Réplica de lectura

:   Copia de la base de datos que acepta consultas de solo lectura.

Failover

:   Conmutación automática a un nodo secundario ante fallo del primario.

Cluster

:   Conjunto de servidores que trabajan juntos.

Balanceo de carga

:   Distribución del tráfico entre varios servidores.

ProxySQL

:   Balanceador y proxy para MySQL.

HAProxy

:   Balanceador TCP/HTTP de alto rendimiento.

pgpool-II

:   Middleware para PostgreSQL con balanceo y pooling.

Patroni

:   Herramienta para alta disponibilidad de PostgreSQL.

AWS RDS

:   Servicio de base de datos relacional gestionado de Amazon.

Migración

:   Proceso de trasladar datos de un sistema a otro.

RTO

:   Recovery Time Objective, tiempo máximo permitido para recuperar el servicio.

RPO

:   Recovery Point Objective, pérdida máxima de datos aceptable.

# Referencias {#referencias .unnumbered}

- Documentación de Docker: <https://docs.docker.com/>

- Documentación de Kubernetes: <https://kubernetes.io/docs/>

- PostgreSQL Streaming Replication: <https://www.postgresql.org/docs/current/warm-standby.html>

- MySQL Group Replication: <https://dev.mysql.com/doc/refman/8.0/en/group-replication.html> MongoDB Replica Sets: <https://docs.mongodb.com/manual/replication/>

- AWS Database Migration Service: <https://aws.amazon.com/dms/>

- Patroni: <https://patroni.readthedocs.io/>

- ProxySQL: <https://proxysql.com/>

- HAProxy: <https://www.haproxy.org/>