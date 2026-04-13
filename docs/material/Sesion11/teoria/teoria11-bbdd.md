---
layout: default
---
# Sesion 11: Cloud Databases - arquitectura, operacion y decision

### 1. Logro de la sesion

Evaluar y disenar despliegues de bases de datos en nube comparando on-premise vs cloud, modelos PaaS vs serverless, y criterios de seguridad, costo, rendimiento y continuidad para soluciones orientadas a analitica e IA.

---

### 2. Contexto de negocio

Las organizaciones migran a cloud por:

- elasticidad,
- reduccion de tiempo de aprovisionamiento,
- servicios gestionados listos para produccion.

No toda carga debe migrar igual; la clave es arquitectura por caso de uso.

---

### 3. On-premise vs cloud

#### 3.1 On-premise

Ventajas:

- control total de infraestructura,
- cumplimiento especifico en entornos regulados.

Desventajas:

- alto CAPEX,
- escalado lento,
- mayor carga operativa interna.

#### 3.2 Cloud

Ventajas:

- escalado rapido,
- menor friccion para pruebas y despliegues,
- servicios administrados.

Desventajas:

- costos variables si no se controla consumo,
- dependencia de proveedor,
- necesidad de buena gobernanza.

---

### 4. Modelos de servicio

#### 4.1 IaaS

Tu gestionas casi todo sobre maquinas virtuales.

#### 4.2 PaaS de base de datos

Proveedor gestiona parches, backups, alta disponibilidad base.

#### 4.3 Serverless database

Escalado automatico por demanda y pago por uso.

---

### 5. PaaS vs Serverless

| Criterio | PaaS | Serverless |
|---|---|---|
| Control fino | Medio | Bajo/medio |
| Escalado | Manual/auto parcial | Automatico |
| Costo en inactividad | Mayor | Menor |
| Latencia fria | Baja | Puede existir |
| Casos tipicos | carga estable | carga variable |

---

### 6. Proveedores principales

#### 6.1 AWS

- RDS,
- Aurora,
- Redshift.

#### 6.2 Azure

- Azure SQL Database,
- Cosmos DB,
- Synapse.

#### 6.3 GCP

- Cloud SQL,
- AlloyDB,
- BigQuery.

---

### 7. Patron de arquitectura recomendado

Separar capas:

1. OLTP transaccional.
2. Analitica/warehouse.
3. Capa de IA y recuperacion.

No sobrecargar una sola base con todos los perfiles de uso.

---

### 8. Disponibilidad y resiliencia

Conceptos clave:

- replicas de lectura,
- multi-zona,
- politicas de backup,
- RPO y RTO.

Definiciones:

- RPO: perdida de datos tolerable.
- RTO: tiempo de recuperacion tolerable.

---

### 9. Seguridad en cloud database

Controles minimos:

- IAM por principio de minimo privilegio,
- cifrado en reposo,
- cifrado en transito TLS,
- rotacion de secretos,
- auditoria y trazabilidad.

---

### 10. Redes y aislamiento

Buenas practicas:

- base en red privada,
- acceso via bastion/VPN cuando aplique,
- evitar exposicion publica innecesaria.

Riesgo clasico:

abrir puertos por urgencia y olvidar cierre.

---

### 11. Costos en nube (FinOps)

Variables principales:

- almacenamiento,
- computo,
- IOPS,
- transferencia de datos.

Estrategias:

1. dimensionar por metricas reales,
2. apagar entornos no productivos fuera de horario,
3. usar retencion y compresion adecuadas.

---

### 12. Observabilidad operacional

Metrica minima por base:

- CPU,
- memoria,
- conexiones activas,
- latencia p95,
- tasa de errores.

Alertas:

- crecimiento anomalo de storage,
- saturacion de conexiones,
- replica con retraso alto.

---

### 13. Migraciones hacia cloud

#### 13.1 Lift and shift

Rapido, pero no siempre optimizado.

#### 13.2 Replatform

Ajusta arquitectura aprovechando servicios gestionados.

#### 13.3 Refactor

Cambio profundo para nativo cloud.

---

### 14. PostgreSQL en cloud

Opciones comunes:

- PostgreSQL administrado (PaaS),
- variantes compatibles de alto rendimiento.

Ventajas:

- ecosistema maduro,
- extensiones como `pg_stat_statements` y `pgvector`,
- SQL estandar y portabilidad razonable.

---

### 15. Cloud y cargas de IA

Escenarios:

- entrenamiento batch en data platform,
- inferencia online con baja latencia,
- RAG con sincronizacion SQL + vectorial.

Requisito:

coordinar base transaccional con almacenamiento analitico y vectorial.

---

### 16. Continuidad y DR

Plan minimo:

1. definir RPO/RTO por sistema,
2. probar restauraciones periodicamente,
3. automatizar backups con verificacion.

Error frecuente:

tener backup sin prueba de restauracion.

---

### 17. Gobernanza multi-entorno

Separar:

- desarrollo,
- QA,
- produccion.

Controlar:

- cambios de esquema,
- accesos privilegiados,
- despliegues con aprobacion.

---

### 18. Caso aplicado

Sistema de tickets IA:

- PostgreSQL PaaS para transacciones,
- almacenamiento analitico para reportes,
- servicio vectorial para busqueda semantica.

Resultado esperado:

- escalado por capa,
- menores riesgos operativos,
- mayor velocidad de evolucion.

---

### 19. Mini laboratorio

1. Elegir entre on-prem y cloud para un caso real.
2. Seleccionar PaaS o serverless y justificar.
3. Definir controles de seguridad minimos.
4. Estimar componentes de costo mensual.
5. Proponer estrategia de backup y DR.

---

### 20. Checklist de dominio

- Distingo CAPEX y OPEX en decision tecnica.
- Selecciono PaaS/serverless segun patron de uso.
- Defino controles de seguridad e IAM.
- Propongo RPO/RTO razonables.
- Identifico riesgos de vendor lock-in.

---

### 21. Preguntas de autoevaluacion

1. Que carga se beneficia mas de serverless?
2. Cuando una replica de lectura no resuelve latencia?
3. Que diferencia hay entre backup y alta disponibilidad?
4. Que practica evita sobrecostos en desarrollo?
5. Por que conviene separar capas OLTP y analitica?

---

### 22. Referencias recomendadas

1. AWS Well-Architected Framework.
2. Azure Architecture Center.
3. Google Cloud Architecture Framework.
4. PostgreSQL on cloud best practices.
5. NIST guidance for cloud security.

