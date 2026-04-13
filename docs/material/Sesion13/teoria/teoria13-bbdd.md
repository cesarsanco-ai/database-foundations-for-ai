---
layout: default
---
# Sesion 13: Bases de datos vectoriales y busqueda semantica

### 1. Logro de la sesion

Comprender el ciclo completo de retrieval semantico para IA generativa: desde embeddings y metrica de similitud hasta indices ANN (HNSW) y uso de motores vectoriales como `pgvector`, diferenciando cuando usar SQL tradicional y cuando usar busqueda por vectores.

---

### 2. Problema que resuelven las vectoriales

SQL clasico responde muy bien a:

- igualdad exacta,
- rangos,
- joins relacionales.

Pero falla cuando la pregunta es semantica:

"busca documentos parecidos a esta idea".

---

### 3. Conceptos base

#### 3.1 Embedding

Representacion numerica (vector) de texto, imagen o audio.

#### 3.2 Dimensionalidad

Cantidad de componentes del vector (ej. 384, 768, 1536).

#### 3.3 Similitud

Medida de cercania entre vectores.

---

### 4. Metricas de similitud

#### 4.1 Cosine similarity

Compara direccion entre vectores.

#### 4.2 Distancia euclidiana

Mide distancia geometrica absoluta.

#### 4.3 Producto punto

Frecuente cuando embeddings estan normalizados.

Regla:

Elegir metrica compatible con el modelo de embeddings.

---

### 5. Exact search vs ANN

#### 5.1 Busqueda exacta

- precisa,
- costosa en volumen alto.

#### 5.2 ANN (Approximate Nearest Neighbors)

- mucho mas rapida,
- leve perdida de exactitud.

En practicas reales de RAG, ANN suele ser estandar.

---

### 6. Indices vectoriales

#### 6.1 HNSW

- alto recall con buena latencia,
- consumo de memoria mayor,
- muy usado en motores modernos.

#### 6.2 IVF (segun motor)

- particiona por clusters,
- configurable para balance precision/velocidad.

---

### 7. `pgvector` en PostgreSQL

Ventaja:

Permite mantener datos relacionales y vectores en una sola plataforma.

Uso base:

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE documentos (
    id BIGSERIAL PRIMARY KEY,
    contenido TEXT NOT NULL,
    embedding VECTOR(1536)
);
```

Busqueda de similitud:

```sql
SELECT id, contenido
FROM documentos
ORDER BY embedding <-> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

---

### 8. Motores vectoriales especializados

Ejemplos:

- Pinecone,
- Milvus,
- Weaviate.

Fortalezas:

- escalado especializado ANN,
- funciones de recuperacion avanzadas.

Trade-off:

mas componentes de arquitectura y sincronizacion.

---

### 9. Pipeline de indexing semantico

1. Ingesta de documentos.
2. Limpieza y chunking.
3. Generacion de embeddings.
4. Almacenamiento vectorial.
5. Indexacion ANN.
6. Evaluacion de retrieval.

---

### 10. Chunking y calidad de recuperacion

Decisiones:

- tamano de chunk,
- solapamiento,
- estrategia por estructura del documento.

Si chunk es muy grande:

- ruido en contexto.

Si chunk es muy pequeno:

- perdida de coherencia semantica.

---

### 11. Filtrado hibrido

Patron recomendado:

- filtrar por metadatos estructurados (fecha, idioma, dominio),
- luego ejecutar similitud vectorial.

En PostgreSQL:

```sql
SELECT id, contenido
FROM documentos
WHERE idioma = 'es'
ORDER BY embedding <-> $1::vector
LIMIT 10;
```

---

### 12. Evaluacion de retrieval

Metricas utiles:

- Recall@k,
- Precision@k,
- MRR (Mean Reciprocal Rank).

No basta evaluar "suena bien"; se necesita evaluacion cuantitativa.

---

### 13. Riesgos comunes

- embeddings desactualizados,
- metadatos incompletos,
- falta de control de versiones de modelo,
- latencia alta por consultas sin filtros,
- no medir calidad de recuperacion.

---

### 14. Seguridad y cumplimiento

Consideraciones:

- cifrado en reposo y transito,
- control de acceso por coleccion/indice,
- tratamiento de PII antes de vectorizar.

Advertencia:

un embedding puede contener señales sensibles del texto original.

---

### 15. Costo y operacion

Componentes de costo:

- generacion de embeddings,
- almacenamiento de vectores,
- consulta ANN.

Estrategias:

1. recalcular embeddings solo cuando cambia contenido,
2. deduplicar documentos,
3. usar politicas de retencion.

---

### 16. SQL relacional vs vectorial

SQL relacional:

- reglas de negocio,
- integridad transaccional,
- reportes estructurados.

Vectorial:

- busqueda semantica,
- recuperacion por similitud,
- soporte a RAG.

Conclusión:

no compiten, se complementan.

---

### 17. Caso aplicado

Asistente academico:

- PostgreSQL guarda cursos, usuarios y permisos.
- Capa vectorial indexa material teorico.
- Consulta final combina seguridad SQL + retrieval ANN.

---

### 18. Mini laboratorio

1. Crear tabla de documentos con `pgvector`.
2. Insertar 20 textos con embeddings de prueba.
3. Ejecutar top-k por similitud.
4. Agregar filtro por categoria.
5. Evaluar manualmente relevancia de resultados.

---

### 19. Checklist de dominio

- Explico embedding y similitud.
- Diferencio exact search y ANN.
- Justifico cuando usar HNSW.
- Integro filtros estructurados y vectoriales.
- Mido calidad con metricas adecuadas.

---

### 20. Preguntas de autoevaluacion

1. Por que ANN es clave en RAG productivo?
2. Que riesgo trae chunking mal calibrado?
3. En que casos `pgvector` es suficiente?
4. Que aporta filtrar por metadatos antes de ANN?
5. Como detectas degradacion de retrieval?

---

### 21. Referencias recomendadas

1. pgvector documentation.
2. Pinecone docs: vector search basics.
3. Milvus docs: ANN and HNSW.
4. Weaviate docs: hybrid search.
5. Papers introductorios de embeddings y retrieval.

