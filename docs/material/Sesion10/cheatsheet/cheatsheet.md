---
layout: default
---

# Cheatsheet: Cloud Databases
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-10](../../../sesiones/sesion-10.md)

---

## 🔥 1. ¿Qué es una Cloud Database?

Una **cloud database** es un servicio de base de datos que se ejecuta en la nube.
No necesitas administrar hardware ni infraestructura; escalas según demanda.

**Ventajas principales:**

* Escalabilidad elástica (horizontal/vertical)
* Alta disponibilidad (multi-AZ)
* Backup y recuperación automáticos
* Seguridad gestionada por el proveedor
* Acceso global vía Internet

**Desventaja:**

* Dependencia del proveedor (vendor lock-in)
* Latencia si se accede desde fuera de la región

---

## 🏗️ 2. Tipos de Cloud Database

| Tipo               | Ejemplo                          | Qué guarda            | Consistencia / Transacciones | Uso típico                      |
| ------------------ | -------------------------------- | --------------------- | ---------------------------- | ------------------------------- |
| Relacional (RDBMS) | Amazon RDS, Cloud SQL, Azure SQL | Tablas                | ACID                         | Apps OLTP, ERP, CRM             |
| NoSQL Document     | MongoDB Atlas, Firebase          | JSON / BSON           | Eventual o ACID parcial      | Apps web, móviles               |
| NoSQL Key-Value    | DynamoDB, Redis                  | Clave → valor         | Eventual                     | Caché, session store            |
| NoSQL Columnar     | Bigtable, Cassandra              | Columnas distribuidas | Eventual                     | Analytics, IoT                  |
| Time Series        | InfluxDB Cloud, Timescale        | Series temporales     | ACID o eventual              | IoT, métricas, logs             |
| Graph              | Neptune, Cosmos DB Graph         | Nodos / aristas       | ACID parcial                 | Redes sociales, recomendaciones |

---

## ⚡ 3. Patrones de Escalabilidad

| Tipo         | Cómo funciona                 | Pros                       | Contras               |
| ------------ | ----------------------------- | -------------------------- | --------------------- |
| Vertical     | Más CPU/RAM en 1 nodo         | Simple                     | Límite físico         |
| Horizontal   | Añadir nodos (sharding)       | Escalable ilimitado        | Complejo              |
| Multi-región | Copias en diferentes regiones | Alta disponibilidad global | Latencia inter-región |

---

## 🧩 4. Modelos de Consistencia

| Modelo   | Qué significa                 | Ejemplo             |
| -------- | ----------------------------- | ------------------- |
| Strong   | Siempre datos correctos       | RDS, Cloud SQL      |
| Eventual | Datos convergen con tiempo    | DynamoDB, Cassandra |
| Causal   | Orden relativo de operaciones | Cosmos DB           |

---

## ☁️ 5. Diferencia Cloud vs On-Premise

| Característica | Cloud               | On-Premise                 |
| -------------- | ------------------- | -------------------------- |
| Gestión HW     | No                  | Sí                         |
| Escalabilidad  | Elástica            | Limitada                   |
| Backup         | Automático          | Manual                     |
| Costo          | Opex (pago por uso) | Capex (inversión inicial)  |
| Disponibilidad | SLA alto            | Depende de infraestructura |

---

## 🛠️ 6. Servicios Populares por Cloud

| Proveedor | Relacional               | NoSQL                | Analítica / Data Warehouse |
| --------- | ------------------------ | -------------------- | -------------------------- |
| AWS       | RDS, Aurora              | DynamoDB, DocumentDB | Redshift                   |
| GCP       | Cloud SQL, Cloud Spanner | Firestore, Bigtable  | BigQuery                   |
| Azure     | Azure SQL                | Cosmos DB            | Synapse Analytics          |

---

## 💡 7. Cloud Database + Big Data

* **Data Lake → almacenamiento masivo (S3, ADLS)**
* **Cloud DB → acceso rápido / transacciones**
* **ETL/ELT → mover datos entre DB y Data Lake**
* **Lakehouse → unifica Data Lake + DW**

---

## 🧠 8. Reglas de Oro / Tips

* 🔹 Apps OLTP → RDBMS en la nube (ACID)
* 🔹 Apps con gran escala / JSON → Document DB
* 🔹 Cargas de analítica → Columnar DB / Data Warehouse
* 🔹 Alta disponibilidad → multi-AZ / replicación
* 🔹 Latencia crítica → nodo cercano al usuario

---

## 🧪 9. Casos de Uso

| Industria | Ejemplo de uso cloud DB                     |
| --------- | ------------------------------------------- |
| Bancos    | Transacciones en RDS / Aurora               |
| Retail    | Catálogos en MongoDB Atlas                  |
| IoT       | Lecturas de sensores en Bigtable / InfluxDB |
| Apps Web  | Sesiones y cache en Redis                   |
| Analytics | Reportes en BigQuery / Redshift             |

---

## 🔄 10. Pipeline típico con Cloud DB

```text
App / IoT → API → Cloud DB → Analytics → Dashboard
```

* **Ejemplo:**
  Usuarios hacen compras → RDS registra → ETL → Redshift → BI

---

## 🎯 11. Resumen ultra corto

* Cloud DB = **Base de datos gestionada en nube**
* Tipos: **Relacional, Document, Key-Value, Columnar, Time Series, Graph**
* Escalabilidad: vertical, horizontal, multi-región
* Consistencia: **Strong / Eventual / Causal**
* Tip: RDBMS = OLTP, Columnar = Analytics, Document = Apps web

