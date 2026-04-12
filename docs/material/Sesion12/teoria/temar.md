# 📘 Temario: Bases de Datos Vectoriales y RAG


## 1. Fundamentos de Búsqueda Semántica
- 1.1. Limitaciones de las bases de datos tradicionales (SQL, NoSQL) para similitud semántica
- 1.2. ¿Qué es un vector en IA? Espacio vectorial, dimensionalidad
- 1.3. Embeddings: definición, generación (texto, imagen, multimodal)
- 1.4. Modelos de embeddings populares:
  - Sentence Transformers, OpenAI embeddings, CLIP, BERT, etc.
  - **Benchmark de modelos**: MTEB Leaderboard, cómo elegir según longitud de texto y dominio
- 1.5. Visualización de embeddings (PCA, t-SNE)
- 1.6. **Semantic Search vs RAG**: diferencias conceptuales y cuándo usar cada uno
- 1.7. Embedding vs LLM: responsabilidades distintas
- 1.8. ¿Es mejor tener mas dimensiones? o ¿cambiar el modelo?
- 1.9. Tipos de modelos: generalistas (MiniLM), Optimized for retrieval (E5,BGE), multimodales, factores de eleccion: Tipo de query (directa vs indirecta), dominio (chat, docs, codigo, etc), longitud de texto, idioma, etc

---

## 2. Métricas de Similitud y Matemática Básica
- 2.1. Similitud de coseno
- 2.2. Distancia euclidiana (L2)
- 2.3. Producto punto
- 2.4. Cuándo usar cada métrica según el modelo y el dominio
- 2.5 Seleccion de modelos de embeddings

---

## 3. Búsqueda de Vecinos Cercanos (k‑NN y ANN)
- 3.1. k‑NN exacto: concepto, complejidad O(n), inviable a gran escala
- 3.2. Approximate Nearest Neighbors (ANN): trade‑off velocidad vs precisión
- 3.3. Algoritmos ANN clave:
  - 3.3.1. HNSW (Hierarchical Navigable Small World): grafos multinivel, estándar de la industria
  - 3.3.2. IVF (Inverted File Index): clustering con centroides
  - 3.3.3. Product Quantization (PQ): compresión de vectores
  - 3.3.4. Comparativa: rendimiento, memoria, precisión

---

## 4. Bases de Datos Vectoriales (Ecosistema Actual)
- 4.1. Bases nativas (Pinecone, Milvus, Weaviate, Qdrant, Chroma)
- 4.2. Extensiones sobre motores existentes:
  - pgvector (PostgreSQL)
  - Elasticsearch (dense vector)
  - Redis Stack / RedisVL
  - MongoDB Atlas Vector Search (tipo `vector` y operador `$vectorSearch`)
- 4.3. Tabla comparativa: nube vs on‑premise, índices soportados, casos de uso

---

## 5. Retrieval Augmented Generation (RAG)
- 5.0. Rol del LLM en RAG (consumo de contexto, limitacines en context window, sensibilidad al orden, capacidad de razonamiento). Interaccion retrieval con LLM
- 5.1. Problema que resuelve (alucinaciones, conocimiento limitado)
- 5.2. Flujo completo: indexación → query → embedding → búsqueda → generación
- 5.3. Orquestadores: LangChain, LlamaIndex, Haystack
- 5.4. **Separación de responsabilidades**: Retrieval vs Generation (el 80% de errores están en retrieval)

---

## 6. Patrones de Producción en RAG (Profundización)

### 6.1. Estrategias de chunking
- Tamaño fijo, overlap, chunking semántico
- Impacto en la calidad de recuperación
- **Ejemplo práctico**: cómo chunkear mensajes de chat vs documentos largos

### 6.2. Metadata enrichment y calidad de datos para embeddings
- Inyección de metadatos (intención, sentimiento, etc.) en el texto a vectorizar
- **Ejemplo antes/después**:
  ```text
  Original: "Reproduce mi playlist favorita"
  Enriquecido: "[INTENCIÓN: reproducir música] El usuario quiere escuchar su playlist favorita"
  ```
- Importancia del **phrasing**: normalización semántica, lenguaje explícito

### 6.3. Optimización del retrieval (core)
- **Alineación semántica Query–Documento**:
  - Diferencia entre pregunta y comando (ej: “¿alguien habló de playlist?” vs “Reproduce mi playlist”)
  - Reformulación de queries (query rewriting)
  - HyDE (Hypothetical Document Embeddings)
- **Top‑K tuning estratégico**:
  - Cómo elegir K: reglas prácticas, trade‑off (bajo recall vs ruido)
  - K dinámico según tipo de query
  - **Retrieval en dos etapas**: ANN (K amplio) + re‑ranking (K final)
- **Re‑ranking con cross‑encoders** (mejora precisión)
- **Hybrid search**: embeddings + BM25 / keyword para evitar fallos semánticos
- **Efecto “Lost in the Middle”**: cómo el orden de los documentos afecta la atención del LLM, sliding window, context packing, truncation inteligente

### 6.4. Memoria en chatbots
- Corto plazo (contexto inmediato)
- Largo plazo (vectorial / RAG)
- Memoria estructurada (perfiles, metadata)

### 6.5. Failure Modes en RAG (basado en experiencia real)
- Documento relevante fuera del Top‑K (bajo recall)
- Embeddings débiles o genéricos
- Chunking incorrecto
- Query mal formulada (indirecta, ambigua)
- Ruido por over‑retrieval
- “Semantic gap”: lo que el usuario dice vs cómo está guardado
- **Depuración vectorial**: inspección de scores de similitud, análisis de falsos positivos/negativos

### 6.6. Retrieval orientado al LLM (avanzado)
- Formateo de contexto para el LLM
- Orden de documentos (priorizar los más relevantes)
- Compresión de contexto (evitar redundancias)
- Evitar contradicciones en el prompt

---

## 7. Integración con SQL y NoSQL
- 7.1. Arquitecturas híbridas: metadatos en SQL, vectores en base vectorial
- 7.2. Estrategias de consistencia: sincrónica, CDC (Debezium, Kafka)
- 7.3. Ejemplo con pgvector: búsqueda híbrida con filtros escalares

---

## 8. Despliegue, Escalabilidad y Costos
- 8.1. On‑premise vs cloud: ventajas y desventajas
- 8.2. Dimensionamiento: número de vectores, dimensionalidad, tipo de índice (HNSW vs IVF)
- 8.3. Costos estimados (almacenamiento, cómputo, operaciones por segundo)
- 8.4. Tuning de índices (nlist, nprobe, efSearch, M) para equilibrio latencia/precisión
- 8.5. **Comportamiento según tamaño del corpus**:
  - Pocos datos → embeddings menos estables, rankings más erráticos
  - Muchos datos → ANN crítico, clustering más útil

---

## 9. Evaluación y Mantenimiento
- 9.1. Métricas de retrieval: recall@K, precision@K, MRR, NDCG
- 9.2. Evaluación de la generación: respuesta correcta, uso del contexto, alucinaciones
- 9.3. Monitoreo en producción: latencia, tasas de error, calidad del contexto recuperado
- 9.4. **Debugging de retrieval** (práctico):
  - Inspección manual de resultados (ej: probar K=3 hasta K=n)
  - Detección de falsos positivos/negativos
  - Tests de regresión: “¿debería aparecer este documento?”
- 9.5 Impacto de seleccion de modelos de embeddings en como cambia el top-k por modelo
- 9.6 Comparacion de embeddings y su impacto

---

## 10. Casos de Uso Reales
- 10.1. Búsqueda semántica en e‑commerce
- 10.2. Sistemas de preguntas y respuestas sobre documentación (FAQ, soporte)
- 10.3. Recomendación de contenidos (streaming, noticias)
- 10.4. Detección de plagio / duplicados
- 10.5. Búsqueda multimodal (texto → imagen, imagen → texto con CLIP)
- 10.6. Que pasa en embedding debil -> mal ranking -> LLM ciego, y no porque sea un LLM pequeño.
- 10.7 Importancia de cambiar embedding en lugar de LLM
---

## 11. Temas Avanzados (Opcional)
- 11.1. Fine‑tuning de embeddings para dominios específicos
- 11.2. Multi‑vector retrieval y Graph RAG
- 11.3. Compresión avanzada: quantization, MRL (Matryoshka Representation Learning)
- 11.4. Streaming y actualizaciones en tiempo real
- 11.5. **LLM-aware retrieval**: optimización del contexto para modelos generativos

---

## 12. Laboratorio Integrador: Debugging de un Fallo Real
- **Objetivo**: replicar el escenario “documento relevante fuera del Top‑K” (caso playlist)
- **Herramientas**: MongoDB Atlas Vector Search, Python, modelo de embeddings pequeño
- **Ejercicios**:
  1. Generar embeddings y configurar índice
  2. Lanzar query y analizar ranking manualmente
  3. Aplicar estrategias:
     - Reformular query (HyDE)
     - Aumentar K + re‑ranking
     - Inyectar metadatos en el texto
  4. Medir mejora con recall@K y similitud visual
  5. Documentar el “failure mode” y la solución

