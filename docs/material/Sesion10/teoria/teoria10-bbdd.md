---
layout: default
---
# Sesion 10: Big Data - formatos, arquitecturas y procesamiento

### 1. Logro de la sesion

Comprender como disenar una plataforma de datos a gran escala diferenciando Data Lake, Data Warehouse y Lakehouse, seleccionando formatos adecuados (especialmente Parquet) y motores de procesamiento para cargas analiticas e IA.

---

### 2. Problema que resuelve Big Data

Cuando crecen volumen, velocidad y variedad:

- el modelo OLTP tradicional no basta,
- aparecen necesidades de historico masivo,
- se requieren pipelines robustos para entrenamiento y analitica.

---

### 3. Las 5V aplicadas

1. Volumen: TB/PB de datos.
2. Velocidad: lotes frecuentes o streaming.
3. Variedad: tabular, logs, texto, audio, imagen.
4. Veracidad: calidad y trazabilidad.
5. Valor: impacto en negocio y producto.

---

### 4. Formatos de datos

#### 4.1 CSV

- simple y portable,
- malo para compresion y tipos,
- util para intercambio basico.

#### 4.2 JSON

- flexible para eventos y APIs,
- mas pesado,
- ideal en ingesta inicial.

#### 4.3 Parquet

- almacenamiento columnar,
- excelente compresion,
- lectura muy eficiente por columnas.

#### 4.4 Avro

- orientado a contratos de datos,
- bueno en streaming.

#### 4.5 ORC

- alto rendimiento analitico en ciertos ecosistemas.

---

### 5. Por que Parquet es clave

Ventajas para analitica:

- pushdown de predicados,
- lectura parcial de columnas,
- mejor compresion que CSV.

Ejemplo practico:

Un dataset de logs en CSV de 500 GB puede reducirse notablemente en Parquet con mejor latencia de lectura.

---

### 6. Arquitecturas de almacenamiento

#### 6.1 Data Warehouse

- datos estructurados y modelados,
- foco en BI y reporting,
- gobernanza fuerte.

#### 6.2 Data Lake

- datos crudos de multiples fuentes,
- costo menor por TB,
- riesgo de "data swamp" sin gobierno.

#### 6.3 Lakehouse

- combina flexibilidad de lake con gobierno de warehouse,
- soporta SQL y workloads de IA sobre una base comun.

---

### 7. Comparativa rapida

| Criterio | Warehouse | Lake | Lakehouse |
|---|---|---|---|
| Tipo de datos | Estructurado | Cualquier tipo | Mixto |
| Gobernanza | Alta | Variable | Alta |
| Costo almacenamiento | Medio/alto | Bajo/medio | Medio |
| Flexibilidad | Media | Alta | Alta |
| SQL analitico | Excelente | Variable | Excelente |

---

### 8. Procesamiento distribuido

#### 8.1 MapReduce

Modelo historico de procesamiento paralelo por lotes.

#### 8.2 Apache Spark

- API de alto nivel,
- SQL, batch y streaming,
- uso masivo en pipelines modernos.

#### 8.3 Motor SQL MPP

Procesamiento paralelo masivo para analitica empresarial.

---

### 9. Batch vs Streaming

#### 9.1 Batch

- procesa volumen acumulado,
- simple de operar,
- mayor latencia.

#### 9.2 Streaming

- procesa eventos continuos,
- menor latencia,
- mayor complejidad operacional.

#### 9.3 Regla practica

Si el negocio tolera horas, batch suele ser suficiente.
Si requiere segundos/minutos, evaluar streaming.

---

### 10. Particionamiento de datos

Particionar por fecha es estandar:

- mejora podado de datos,
- reduce costo de consulta.

Ejemplo de ruta:

`/eventos/year=2026/month=04/day=13/`

Buenas practicas:

- evitar demasiadas particiones pequenas,
- alinear particion con filtros frecuentes.

---

### 11. Calidad y gobernanza en Big Data

Sin gobierno, el lago se degrada.

Elementos minimos:

- catalogo de datos,
- linaje,
- validaciones de calidad,
- ownership por dominio.

---

### 12. Modelado para analitica e IA

#### 12.1 Tablas de hechos y dimensiones

- hechos: eventos transaccionales,
- dimensiones: contexto descriptivo.

#### 12.2 Feature store

Capa para servir variables de ML consistentes entre entrenamiento e inferencia.

#### 12.3 Relacion curricular

Conecta con semana ETL y con semana de arquitectura hibrida para RAG.

---

### 13. PostgreSQL en ecosistema Big Data

PostgreSQL no reemplaza todo el stack Big Data, pero aporta:

- staging y control transaccional,
- transformaciones SQL robustas,
- fuente de verdad para entidades core.

Conexion con scripts ETL:

- uso de schemas `bronze/silver/gold`,
- procedimientos para mover y limpiar datos.

---

### 14. Patrón medallon (Bronze, Silver, Gold)

#### 14.1 Bronze

Datos crudos, historico completo.

#### 14.2 Silver

Datos limpios y estandarizados.

#### 14.3 Gold

Datos agregados para negocio.

Ventaja:

Separar responsabilidades y reducir acoplamiento.

---

### 15. Costos y finops de datos

Variables de costo:

- almacenamiento,
- computo de transformacion,
- transferencia entre regiones.

Principios finops:

1. medir costo por dataset,
2. automatizar borrado/retencion,
3. priorizar formatos eficientes.

---

### 16. Seguridad en plataformas Big Data

Controles minimos:

- IAM por rol,
- cifrado en reposo y transito,
- mascarado de datos sensibles,
- auditoria de acceso.

---

### 17. Errores frecuentes

- usar CSV como formato final analitico,
- no versionar definiciones de datasets,
- mezclar datos crudos y curados en la misma zona,
- sobreparticionar y crear archivos pequenos,
- ignorar calidad en ingesta.

---

### 18. Caso de estudio

Escenario:

Plataforma de mensajeria IA con millones de eventos diarios.

Decisiones:

- ingestar eventos crudos en lake,
- transformar con SQL/Spark a capa silver,
- exponer gold para dashboards y modelos.

Resultado:

- menor tiempo de analisis,
- trazabilidad completa,
- mejor experiencia para equipo de datos.

---

### 19. Mini laboratorio

1. Definir un dataset de eventos y su esquema logico.
2. Diseñar particion por fecha.
3. Elegir formato de almacenamiento final.
4. Proponer flujo Bronze -> Silver -> Gold.
5. Explicar por que no dejar todo en una sola tabla.

---

### 20. Checklist de dominio

- Distingo warehouse, lake y lakehouse.
- Explico por que Parquet acelera analitica.
- Justifico batch o streaming segun SLA.
- Diseno particiones sin sobrefragmentar.
- Relaciono Big Data con pipelines de IA.

---

### 21. Preguntas de autoevaluacion

1. Que diferencia central hay entre lake y warehouse?
2. Cuando Parquet no es la mejor eleccion?
3. Que problema genera un "data swamp"?
4. Que capa del patron medallon consume BI?
5. Como reducir costo sin perder calidad?

---

### 22. Referencias recomendadas

1. The Data Warehouse Toolkit.
2. Designing Data-Intensive Applications.
3. Apache Spark Documentation.
4. Delta Lake / Iceberg documentation.
5. Guías de arquitectura de datos en nube.

