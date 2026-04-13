---
layout: default
---
# Sesion 1: El universo del dato y la infraestructura de bases de datos

### 1. Logro de la sesion

Comprender de forma integral como se transforma un dato crudo en un activo util para negocio e IA, relacionando fundamentos de informacion, arquitectura fisica, complejidad de busqueda, roles profesionales, gobernanza y tendencias modernas (RAG y bases vectoriales), para tomar decisiones de diseno con criterio tecnico.

---

### 2. Fundamentos conceptuales: dato, informacion, conocimiento

#### 2.1 La cadena de valor del dato

Un sistema de bases de datos no administra "magia", administra niveles de abstraccion:

- **Dato:** hecho crudo sin contexto.
- **Informacion:** dato procesado con significado en un contexto.
- **Conocimiento:** informacion interpretada para decidir y actuar.

**Ejemplo rapido**

- Dato: `1010`, `2026-04-10`, `79.90`.
- Informacion: el cliente 1010 compro un producto por 79.90 el 10 de abril.
- Conocimiento: ese cliente compra mensualmente y es candidato a fidelizacion.

#### 2.2 Jerarquia DIKW aplicada a BBDD

| Nivel | Pregunta que responde | Ejemplo en sistemas |
|------|------------------------|---------------------|
| Dato | Que paso | Registro de una transaccion |
| Informacion | Que significa | Dashboard con ventas por region |
| Conocimiento | Que hago ahora | Activar una promocion por caida de conversion |

#### 2.3 Tipos de datos por estructura

- **Estructurados:** esquema fijo (tablas relacionales).
- **Semiestructurados:** estructura flexible con etiquetas (JSON, XML).
- **No estructurados:** texto libre, audio, imagen, video.

**Regla de ingenieria:** cuanto mas flexible es el dato, mayor esfuerzo posterior en limpieza, catalogacion y recuperacion.

---

### 3. Historia y evolucion de los sistemas de datos

#### 3.1 Linea temporal sintetica

| Periodo | Hito | Impacto |
|--------|------|---------|
| 1960s | Sistemas jerarquicos y de red | Primeros motores de almacenamiento empresarial |
| 1970 | Modelo relacional (E. F. Codd) | Nace la base teorica de SQL |
| 1980s | SQL comercial | Estandarizacion de consultas y transacciones |
| 1990s | Data warehouse y BI | Analitica historica empresarial |
| 2000s | Big Data y NoSQL | Escalabilidad horizontal y nuevos modelos |
| 2010s | Cloud data platforms | Elasticidad, servicios administrados |
| 2020s | Vector DB y RAG | Recuperacion semantica para IA generativa |

#### 3.2 Del OLTP a AI-native data stacks

Antes, la prioridad era registrar transacciones de forma confiable (OLTP).  
Hoy, ademas de transacciones, se exige:

- analitica en tiempo casi real,
- entrenamiento e inferencia de modelos,
- trazabilidad y gobernanza multi-equipo.

**Conclusion historica:** la base de datos dejo de ser solo "almacen", ahora es una capa central de producto y decision.

---

### 4. Formatos, tipos y codificacion: decisiones que impactan rendimiento

#### 4.1 Tipos de datos basicos en motores relacionales

- Numericos: `INT`, `BIGINT`, `DECIMAL`, `FLOAT`.
- Texto: `CHAR`, `VARCHAR`, `TEXT`.
- Temporales: `DATE`, `TIME`, `TIMESTAMP`.
- Booleanos: `BOOLEAN`.
- Binarios: `BLOB`.

**Buenas practicas**

1. Usar `DECIMAL` en dinero para evitar errores de redondeo.
2. Evitar `TEXT` cuando una longitud acotada permite indexacion mas eficiente.
3. Definir zonas horarias y convencion temporal desde el inicio.

#### 4.2 Codificacion de caracteres

La codificacion incorrecta rompe integridad semantica (acentos, simbolos, emojis, idiomas mixtos).

- ASCII: limitado.
- ISO-8859-1: heredado.
- UTF-8: estandar recomendado.

**Recomendacion docente:** usar UTF-8 de extremo a extremo (ingesta, almacenamiento y exportacion).

#### 4.3 Formatos de archivo en pipelines de datos

| Formato | Ventaja principal | Limite principal | Caso de uso |
|--------|-------------------|------------------|-------------|
| CSV | Portabilidad | Sin esquema robusto | Intercambio simple |
| JSON | Flexibilidad | Verboso | APIs y documentos |
| Parquet | Lectura columnar eficiente | Menos ideal para escritura fila a fila | Data lake y analitica |
| Avro | Evolucion de esquema | No legible humano | Streaming y contratos de datos |
| ORC | Alta compresion | Menor adopcion fuera de ecosistemas concretos | Workloads analiticos |

**Ejemplo practico**

Un dataset de 1 TB en CSV puede bajar a ~200-300 GB en Parquet comprimido y acelerar consultas por columnas especificas.

---

### 5. Metadata, catalogos y linaje

#### 5.1 Que es metadata

Metadata es "dato sobre el dato". Sin metadata no hay descubrimiento confiable ni gobernanza real.

Tipos clave:

1. **Descriptiva:** nombre, descripcion, propietario.
2. **Estructural:** columnas, tipos, particiones.
3. **Operativa:** origen, fecha de actualizacion, SLA.
4. **Governance:** clasificacion de sensibilidad, retencion, politicas.

#### 5.2 Data lineage

El linaje responde:

- de donde vino este campo,
- que transformaciones sufrio,
- quien lo modifico y cuando.

**Caso util para estudiar**

Si una metrica de conversion cae abruptamente, lineage permite ubicar si el problema esta en:

- fuente de eventos,
- transformacion ETL,
- modelo semantico del dashboard.

#### 5.3 Data catalogs y documentacion minima

Cada dataset deberia tener:

- objetivo de negocio,
- granularidad (evento, cliente, sesion),
- periodicidad de actualizacion,
- reglas de calidad,
- contacto responsable.

---

### 6. Infraestructura fisica y jerarquia de memoria

#### 6.1 Idea central

La latencia domina la experiencia de consulta.  
En bases de datos, cada salto de memoria cuesta tiempo.

| Capa | Latencia aproximada | Uso tipico |
|------|---------------------|------------|
| Cache CPU | ns | Operaciones inmediatas |
| RAM / Buffer pool | decenas-cientos ns | Paginas calientes |
| NVMe / SSD | decenas-cientos us | Persistencia rapida |
| HDD | ms | Almacen economico legado |
| Red remota / SAN | ms adicionales | Infra compartida |

#### 6.2 Buffer pool y cache hit ratio

El motor intenta mantener paginas consultadas frecuentemente en memoria.

- **Cache hit alto** -> consultas rapidas.
- **Cache miss alto** -> mas I/O y latencia.

**Regla practica:** en sistemas transaccionales, un hit ratio elevado suele correlacionar con estabilidad operacional.

#### 6.3 I/O y patrones de acceso

- Acceso secuencial: mejor para lecturas masivas.
- Acceso aleatorio: mas costoso en discos mecanicos.
- SSD/NVMe reduce penalizacion, pero no elimina necesidad de buen diseno.



### 7. Dimensionamiento y arquitectura de datos

#### 7.1 Escala de volumen

- GB: proyectos pequenos.
- TB: analitica empresarial estandar.
- PB: plataformas de alto trafico.

#### 7.2 Las 5V en sistemas de datos modernos

1. **Volumen**
2. **Velocidad**
3. **Variedad**
4. **Veracidad**
5. **Valor**

No basta con almacenar: hay que servir datos confiables a la velocidad necesaria para decision y producto.

#### 7.3 Patrones arquitectonicos comunes

| Patron | Descripcion | Fortaleza |
|-------|-------------|-----------|
| OLTP relacional | Escritura transaccional | Integridad y consistencia |
| Data warehouse | Analitica historica | Consultas agregadas complejas |
| Data lake | Almacenamiento bruto escalable | Flexibilidad y costo |
| Lakehouse | Mezcla lake + warehouse | Unificacion analitica moderna |
| Arquitectura hibrida SQL + vectorial | SQL para negocio, vector para recuperacion semantica | Casos de IA generativa |

---

### 8. Roles profesionales y colaboracion en equipos de datos

#### 8.1 Roles clave

- **DBA:** disponibilidad, seguridad, tuning, backups.
- **Data Engineer:** pipelines, modelos de datos, orquestacion.
- **Analytics Engineer:** capa semantica y modelos analiticos.
- **Data Scientist / ML Engineer:** features, entrenamiento, despliegue.
- **Data Steward:** calidad, definiciones, gobernanza.

#### 8.2 Fricciones frecuentes

1. Definiciones inconsistentes de metricas.
2. Falta de ownership por tabla o dominio.
3. Cambios de esquema sin comunicacion.
4. Ausencia de pruebas de datos.

#### 8.3 Practicas para evitar caos

- Contratos de datos (data contracts).
- Versionado de transformaciones.
- Revisiones de cambios de esquema.
- Alertas por calidad y freshness.

---

### 9. Gobernanza, seguridad y etica de datos

#### 9.1 Gobernanza practica

Gobernanza no es burocracia, es capacidad de operar con confianza:

- reglas de acceso por rol,
- politicas de retencion,
- trazabilidad de cambios,
- controles de calidad automatizados.

#### 9.2 Seguridad esencial

| Control | Objetivo | Ejemplo |
|--------|----------|---------|
| Autenticacion | Verificar identidad | SSO, MFA |
| Autorizacion | Limitar acciones | RBAC por esquema o tabla |
| Cifrado en reposo | Proteger almacenamiento | AES-256 en discos |
| Cifrado en transito | Proteger trafico | TLS entre servicios |
| Auditoria | Evidencia forense | Logs de accesos y cambios |

#### 9.3 Etica y sesgo

Si los datos historicos estan sesgados, el modelo aprende y replica ese sesgo.

Mitigaciones iniciales:

- revision de representatividad de muestras,
- seguimiento de metricas por subgrupo,
- explicabilidad de decisiones en dominios sensibles.

---

### 10. El desafio moderno: IA, contexto y bases vectoriales

#### 10.1 Limite de ventana de contexto

Un LLM no puede "cargar" toda la base de conocimiento empresarial en el prompt:

- costo computacional creciente,
- latencia,
- riesgo de desactualizacion.

#### 10.2 RAG como patron de arquitectura

`Pregunta -> embedding -> recuperacion -> contexto relevante -> generacion`

Elementos principales:

1. Fuente documental confiable.
2. Proceso de chunking coherente.
3. Embeddings de calidad.
4. Indice vectorial.
5. Evaluacion continua de recuperacion.

#### 10.3 SQL + vector: por que arquitectura hibrida

- SQL conserva consistencia transaccional y reglas de negocio.
- Vector DB optimiza similitud semantica para lenguaje natural.

**Ejemplo de uso**

Un asistente interno consulta politicas de RRHH:

- usa SQL para validar permisos del empleado,
- usa recuperacion vectorial para encontrar fragmentos relevantes,
- responde con evidencia citada.

---


---

### 11. Referencias recomendadas

1. Codd, E. F. (1970). A Relational Model of Data for Large Shared Data Banks.
2. Kleppmann, M. (2017). Designing Data-Intensive Applications.
3. Silberschatz, A., Korth, H. F., Sudarshan, S. Database System Concepts.
4. Kimball, R., Ross, M. The Data Warehouse Toolkit.
5. Huyen, C. (2022). Designing Machine Learning Systems.
6. Documentacion oficial de PostgreSQL y MySQL (indices, planner y tuning).
7. Documentacion de pgvector, Milvus o Weaviate para recuperacion semantica.


