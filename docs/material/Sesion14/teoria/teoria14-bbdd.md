---
layout: default
---
# Sesion 14: Arquitecturas hibridas SQL + vectorial para RAG

### 1. Logro de la sesion

Disenar una arquitectura hibrida para IA generativa que combine PostgreSQL, motores vectoriales y orquestacion de aplicaciones (LangChain/LlamaIndex), incorporando sincronizacion de datos, observabilidad y controles de calidad para sistemas RAG en produccion.

---

### 2. Vision integral de la sesion

Capas del sistema:

1. Datos transaccionales en SQL.
2. Indexacion semantica vectorial.
3. Orquestacion del flujo de pregunta-respuesta.
4. Monitoreo, evaluacion y gobierno.

---

### 3. Que es RAG

RAG (Retrieval Augmented Generation) combina:

- recuperacion de contexto relevante,
- generacion con modelo de lenguaje.

Pipeline simplificado:

`Pregunta -> embedding -> retrieval -> re-rank -> prompt -> respuesta`.

---

### 4. Por que arquitectura hibrida

Un solo motor no resuelve todo:

- SQL garantiza reglas de negocio y permisos,
- vectorial aporta busqueda semantica,
- orquestador coordina pasos y herramientas.

---

### 5. Componentes tecnicos

#### 5.1 Base SQL (PostgreSQL)

- usuarios,
- roles,
- entidades del negocio,
- logs de interaccion.

#### 5.2 Vector store

- chunks,
- embeddings,
- metadatos de documento.

#### 5.3 Orquestacion

- LangChain o LlamaIndex,
- control de prompts y herramientas,
- evaluacion de cadenas.

---

### 6. Flujo de una consulta RAG

1. Validar identidad y permisos en SQL.
2. Normalizar consulta del usuario.
3. Generar embedding de la consulta.
4. Recuperar top-k chunks en indice vectorial.
5. Filtrar por metadatos y politicas.
6. Construir prompt con contexto.
7. Generar respuesta.
8. Registrar trazas y feedback.

---

### 7. Sincronizacion SQL <-> vectorial

Problema:

cuando cambia documento fuente, embedding queda obsoleto.

Opciones:

- batch nocturno,
- micro-batch continuo,
- CDC (Change Data Capture).

Regla:

definir SLA de frescura segun criticidad del dominio.

---

### 8. CDC en arquitecturas hibridas

CDC permite detectar:

- inserciones,
- actualizaciones,
- eliminaciones.

Uso en RAG:

- reindexar chunks afectados,
- mantener consistencia entre catalogo SQL y vector store.

---

### 9. Estrategias de chunking y versionado

Guardar siempre:

- `document_id`,
- `chunk_id`,
- `version`,
- `hash_contenido`.

Beneficio:

reindexacion incremental confiable y auditable.

---

### 10. Seguridad en RAG empresarial

Controles clave:

1. ACL por documento.
2. Filtrado de retrieval por permisos.
3. Evitar exponer secretos en contexto.
4. Registro de acceso a fragmentos sensibles.

Sin esto, un RAG puede filtrar informacion no autorizada.

---

### 11. Calidad de respuesta

Dimensiones:

- relevancia factual,
- cobertura de contexto,
- ausencia de alucinacion,
- citabilidad de fuentes.

Metricas:

- hit rate de retrieval,
- faithfulness,
- precision de respuesta por dominio.

---

### 12. Observabilidad y trazabilidad

Monitorear:

- latencia por etapa,
- tasa de fallos,
- consultas sin contexto util,
- costo por request.

Registrar:

- pregunta,
- documentos recuperados,
- prompt final,
- respuesta final,
- score de confianza.

---

### 13. Manejo de costos

Costo total de un RAG incluye:

- embeddings,
- almacenamiento vectorial,
- inferencia LLM,
- observabilidad.

Optimizar:

1. caching de consultas frecuentes,
2. top-k ajustado por dominio,
3. prompt compacto y util.

---

### 14. Patrones de despliegue

#### 14.1 Monolito inicial

Rapido para pilotos.

#### 14.2 Servicios separados

Mejor escalado por componente.

#### 14.3 Arquitectura orientada a eventos

Recomendada cuando hay CDC y reindexacion continua.

---

### 15. Integracion con LangChain/LlamaIndex

Casos de uso:

- pipelines de ingestion,
- retrieval hibrido,
- evaluacion automatizada.

Criterio:

usar framework cuando reduce complejidad real, no por moda.

---

### 16. PostgreSQL en arquitectura hibrida

Rol de PostgreSQL:

- fuente de verdad transaccional,
- gobierno de usuarios y permisos,
- almacenamiento de telemetria de interacciones,
- opcion de vector search con `pgvector` en escenarios medianos.

---

### 17. Riesgos de implementacion

- no definir ownership de datos,
- sincronizacion eventual sin monitoreo,
- falta de pruebas de seguridad por rol,
- observabilidad incompleta,
- ausencia de evaluacion continua de calidad.

---

### 18. Caso de estudio integrador

Asistente legal interno:

- SQL guarda clientes, expedientes y permisos.
- Vectorial indexa contratos y jurisprudencia.
- Orquestador aplica filtros de confidencialidad antes de responder.

Resultado esperado:

- respuestas utiles,
- trazables,
- alineadas a politicas de acceso.

---

### 19. Plan de adopcion por fases

Fase 1:

- PoC con dominio acotado.

Fase 2:

- medicion formal de retrieval y calidad.

Fase 3:

- hardening de seguridad y costos.

Fase 4:

- escalado multi-dominio y operaciones continuas.

---

### 20. Checklist de produccion

- Defini SLA de frescura de indice.
- Implementé control de acceso en retrieval.
- Tengo trazabilidad extremo a extremo.
- Mido latencia y calidad por release.
- Existe plan de rollback de indice/prompts.

---

### 21. Mini laboratorio final

1. Modelar tablas SQL para usuarios, documentos y permisos.
2. Definir esquema de chunks y embeddings.
3. Simular una sincronizacion incremental.
4. Diseñar consulta RAG con filtro de acceso.
5. Evaluar tres respuestas con rubrica de calidad.

---

### 22. Preguntas de autoevaluacion

1. Por que RAG requiere SQL y vectorial a la vez?
2. Que problema resuelve CDC en esta arquitectura?
3. Que metrica detecta degradacion del retrieval?
4. Que riesgo hay si el filtro de permisos va al final?
5. Cuando conviene usar `pgvector` y no un motor separado?

---

### 23. Cierre del curso

Competencias integradas logradas:

- modelado relacional,
- SQL analitico y optimizacion,
- programacion en servidor,
- ETL y arquitectura de datos,
- NoSQL y vectoriales,
- diseno de sistemas RAG empresariales.

Proximo paso recomendado:

construir un proyecto capstone con datos reales y evaluacion cuantitativa.

---

### 24. Referencias recomendadas

1. Lewis et al. (2020), Retrieval-Augmented Generation.
2. Documentacion LangChain.
3. Documentacion LlamaIndex.
4. pgvector documentation.
5. Guías de arquitectura RAG en produccion.

