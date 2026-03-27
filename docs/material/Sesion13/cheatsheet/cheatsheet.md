---
layout: default
---

# Cheatsheet: Arquitecturas hibridas
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-13](../../../sesiones/sesion-13.md)

---

## 1. Modelos de Despliegue: On-Premise vs. Cloud

La elección entre On-Premise (infraestructura local) y Cloud (nube) define la base sobre la que se construye la gestión de datos. En la práctica, la mayoría de las grandes organizaciones optan por un **modelo híbrido** que combina ambos.

| Característica | On-Premise | Cloud (IaaS/PaaS) | Modelo Híbrido |
| :--- | :--- | :--- | :--- |
| **Control** | Total sobre hardware, software y seguridad. | Limitado por el proveedor (AWS, Azure, GCP). | Balance: crítico on-prem, elástico en cloud. |
| **Escalabilidad** | Limitada por capacidad física; requiere compra de hardware. | Elástica y bajo demanda (auto-scaling). | Escala en cloud para picos; base on-prem para carga constante. |
| **Costo** | Alto CAPEX (hardware, espacio, electricidad). | OPEX (pago por uso). | Optimización: reservas en cloud + amortización on-prem. |
| **Latencia** | Mínima (red local). | Variable; depende de la región y la red. | Crítico en borde local; analítica en cloud. |
| **Seguridad** | Responsabilidad 100% interna. | Responsabilidad compartida (proveedor gestiona la nube, tú gestionas el acceso y los datos). | Zonas desmilitarizadas (DMZ) y VPC peering. |
| **Casos de Uso** | Datos sensibles, legacy, regulación estricta (banca, salud). | Startups, big data, disaster recovery, desarrollo ágil. | **Burstable workloads**: usar cloud para picos de demanda y on-prem para la carga base. |

> **Conclusión Clave**: *On-premise* es el "banco de datos" seguro pero rígido. *Cloud* es el "centro logístico" elástico pero con costos operativos. **La arquitectura híbrida une ambos mundos.**

---

## 2. Arquitecturas Híbridas de Bases de Datos (Políglota)

Una **arquitectura híbrida** (no confundir con despliegue híbrido) combina diferentes motores de bases de datos para aprovechar las fortalezas de cada uno. Es la evolución natural de los entornos cloud y on-prem.

### 2.1. Componentes y Estrategias de Despliegue

| Capa | Motor | Rol | Estrategia On-Premise | Estrategia Cloud | Estrategia Híbrida |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Transaccional (SQL)** | PostgreSQL, Oracle, SQL Server | Datos críticos, ACID, maestros. | Cluster físico con SAN y alta disponibilidad local. | **RDS/Aurora** (AWS), **Cloud SQL** (GCP), **Azure SQL**. | **Active-Active**: Maestro on-prem, réplica en cloud para DR y lectura. |
| **Velocidad (NoSQL)** | Redis, MongoDB, Cassandra | Caché, sesiones, datos semiestructurados. | Servidores dedicados con mucha RAM. | **ElastiCache** (Redis/Memcached), **DocumentDB**, **Cosmos DB**. | Cache local para baja latencia; sincronización asíncrona a cloud para análisis. |
| **Memoria Semántica (Vectorial)** | Milvus, Pinecone, pgvector | Embeddings, RAG, búsqueda semántica. | Clusters con GPUs/NVMe para indexación. | **Pinecone**, **Weaviate Cloud**, **Vertex AI Vector Search**. | Indexación intensiva en cloud (GPU), consultas de baja latencia en borde on-prem. |

### 2.2. Patrón de Arquitectura Híbrida

```mermaid
graph TD
    subgraph On-Premise
        A[Usuarios/Apps] --> B[(PostgreSQL\nTransaccional)]
        B --> C[(Redis\nCaché Local)]
        D[Sensores IoT] --> E[(Milvus\nVectorial Local)]
    end

    subgraph Cloud (AWS/Azure/GCP)
        F[(Aurora/RDS\nRéplica Lectura)]
        G[(ElastiCache\nCaché Global)]
        H[(Pinecone/pgvector\nIndexación Vectorial)]
        I[Data Warehouse\n(BigQuery/Snowflake)]
    end

    B -- CDC (Debezium/Kafka) --> F
    F -- ETL --> I
    B -- Embeddings API --> H
    E -- Sync Batch --> H
    C <---> G
```

---

## 3. Integración y Sincronización en Entornos Híbridos

### 3.1. Conectividad Segura
- **VPN o Direct Connect (AWS) / ExpressRoute (Azure)**: Establece un enlace privado y de baja latencia entre el centro de datos on-prem y la nube.
- **VPC Peering**: Permite la comunicación entre diferentes entornos de nube (ej. AWS a GCP) si se utiliza multicloud.

### 3.2. Sincronización de Datos (On-Prem ↔ Cloud)

| Método | Descripción | Uso Típico |
| :--- | :--- | :--- |
| **Change Data Capture (CDC)** | Herramientas como **Debezium** o **AWS DMS** capturan cambios del WAL (Write-Ahead Log) on-prem y los replican en tiempo real a la nube. | Mantener una réplica activa en cloud para Disaster Recovery (DR) o para descargar consultas analíticas. |
| **ETL/ELT Batch** | Procesos programados (ej. cada hora) que mueven datos desde on-prem a un Data Lake/ Warehouse en la nube. | Consolidación diaria de ventas, cargas históricas para entrenamiento de IA. |
| **Triggers y Doble Escritura** | La aplicación escribe simultáneamente en la DB on-prem y envía eventos a una cola (Kafka) para procesamiento en cloud. | Sistemas de baja tolerancia a la latencia en la consistencia (suele ser complejo). |

### 3.3. Gestión de Consistencia
- **Consistencia Fuerte**: Para transacciones financieras, se mantiene el maestro en on-prem con replicación síncrona a cloud (aumenta latencia).
- **Consistencia Eventual**: Para búsquedas vectoriales y cachés, se asume que los datos en cloud pueden tener segundos o minutos de retraso respecto al on-prem.
- **Data Residency**: Es crucial para cumplir normativas (GDPR) que los datos sensibles nunca salgan de la región on-prem o de una zona específica de la nube.

---

## 4. Orquestación y Gestión

### 4.1. Infraestructura como Código (IaC)
- **Terraform**: Es la herramienta estándar para gestionar recursos tanto on-prem (vía vSphere, OpenStack) como en cloud (AWS, Azure, GCP) de manera unificada.
- **Beneficio**: Permite crear entornos de desarrollo idénticos a producción (dev/prod parity) y realizar "drills" de disaster recovery con solo cambiar variables.

### 4.2. Contenedores y Kubernetes
- **On-Prem**: Kubernetes on-prem (OpenShift, Rancher, vanilla K8s) permite estandarizar el despliegue de bases de datos en contenedores con **StatefulSets** y almacenamiento persistente local.
- **Cloud**: Servicios gestionados como **EKS** (AWS), **AKS** (Azure), **GKE** (GCP) eliminan la sobrecarga de gestión del plano de control.
- **Híbrido**: **Anthos** (GCP) o **Azure Arc** permiten gestionar clusters Kubernetes on-prem desde la consola de la nube, unificando políticas y monitoreo.

---

## 5. Casos de Uso: Cuándo Usar Cada Modelo

| Escenario | Modelo Recomendado | Justificación |
| :--- | :--- | :--- |
| **Base de datos de clientes (PII) en banco europeo** | **On-Premise** | Regulación estricta (GDPR) exige control total sobre ubicación de datos personales. |
| **Startup con IA generativa en fase de crecimiento** | **Cloud (PaaS)** | Escalabilidad inmediata, acceso a GPUs/TPUs y servicios de base de datos vectorial gestionados. |
| **Empresa con picos estacionales (ej. Black Friday)** | **Híbrido (Burstable)** | On-prem maneja la carga base (70%). Cloud escala automáticamente para absorber el pico del 30% restante, pagando solo por lo usado. |
| **Disaster Recovery (DR)** | **Híbrido (Pilot Light)** | On-prem como principal. Réplica en cloud (ej. AWS) con datos replicados vía CDC. En caso de fallo, se "enciende" la infraestructura en cloud en minutos. |
| **Entrenamiento de Modelos ML** | **Cloud (Data Lake)** | Mover grandes volúmenes de datos (Parquet) al cloud para usar motores de cómputo distribuido (Spark) sin saturar la red on-prem. |

---

## 6. Retos y Buenas Prácticas en Entornos Híbridos

1.  **Latencia de Red**: La distancia física entre on-prem y cloud es enemiga de la consistencia fuerte.
    - *Solución*: Usar **Direct Connect** y diseñar aplicaciones para ser *cloud-aware* (escribir local, leer local siempre que sea posible).
2.  **Gestión de Secretos**: Las credenciales no deben estar hardcodeadas ni en on-prem ni en cloud.
    - *Solución*: Usar **HashiCorp Vault** (on-prem) integrado con **AWS Secrets Manager** o **Azure Key Vault**.
3.  **Costos Inesperados**: La salida de datos (*egress*) de cloud hacia on-prem suele tener costo.
    - *Solución*: Optimizar el flujo de datos; preferir que el consumo de datos se realice dentro de la misma región de cloud si es posible.
4.  **Split-Brain (División de cerebro)** : Durante una caída de red, ambos lados (on-prem y cloud) podrían intentar actuar como maestros.
    - *Solución*: Implementar mecanismos de *lease* o *quorum* (ej. usando ZooKeeper/Etcd) para definir quién es el "dueño" de la escritura en caso de partición de red.

---

## 7. Resumen: Matriz de Decisión

| Requisito | On-Premise | Cloud | Híbrido |
| :--- | :--- | :--- | :--- |
| **Latencia Ultra Baja** | ✅ (Red local) | ❌ (Depende de internet) | ✅ (Core local, borde en cloud) |
| **Escalabilidad Instantánea** | ❌ (Lenta) | ✅ (Auto-scaling) | ✅ (Elástico para picos) |
| **Cumplimiento Normativo Estricto** | ✅ (Control total) | ⚠️ (Requiere configuración avanzada) | ✅ (Datos sensibles local, procesos en cloud) |
| **CAPEX Controlado** | ❌ (Inversión inicial alta) | ✅ (Pago por uso) | ✅ (Optimización de recursos) |
| **Resiliencia y DR** | ⚠️ (Requiere segundo centro de datos) | ✅ (Multi-AZ por defecto) | ✅ (Cloud como DR site) |

---

**Conclusión Final**: La gestión de datos moderna ya no se trata de elegir entre *on-premise* o *cloud*, sino de orquestar una **arquitectura híbrida** que combine **SQL para la verdad transaccional**, **NoSQL para la velocidad**, **Vectorial para la semántica (IA)** , y **On-Prem + Cloud para el control y la elasticidad**. El éxito radica en la correcta implementación de la conectividad, la sincronización (CDC) y la gobernanza unificada.