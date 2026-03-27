---
layout: default
---

# Cheatsheet: Bases de datos vectoriales
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-12](../../../sesiones/sesion-12.md)

---

## 1️⃣ Conceptos Fundamentales

| Concepto                  | Definición / Uso                                                                                         |
| ------------------------- | -------------------------------------------------------------------------------------------------------- |
| Vector / Embedding        | Representación numérica de un objeto (texto, imagen, audio) en un espacio de alta dimensión.             |
| Dimensionalidad           | Número de componentes del vector (ej. 384, 768, 1536).                                                   |
| Similitud de coseno       | Mide cercanía semántica ignorando magnitud; valor cercano a 1 = similar.                                 |
| Distancia euclidiana (L2) | Distancia geométrica tradicional entre vectores.                                                         |
| Producto punto            | Similar a coseno si vectores están normalizados.                                                         |
| ANN                       | Approximate Nearest Neighbors: búsqueda rápida aproximada de vecinos cercanos.                           |
| RAG                       | Retrieval Augmented Generation: combinación de recuperación de datos + LLM para respuestas contextuales. |

---

## 2️⃣ Algoritmos de Búsqueda Vectorial

| Algoritmo | Características             | Velocidad | Precisión                    |
| --------- | --------------------------- | --------- | ---------------------------- |
| HNSW      | Grafos multinivel           | Alta      | Muy alta                     |
| IVF       | Clusters con k-means        | Alta      | Alta (depende de n_clusters) |
| PQ        | Cuantización de subvectores | Muy alta  | Media                        |

---

## 3️⃣ Tecnologías de Bases de Datos Vectoriales

| Tecnología    | Tipo                 | Índices soportados | Despliegue         |
| ------------- | -------------------- | ------------------ | ------------------ |
| Pinecone      | Nativa gestionada    | HNSW               | Solo cloud         |
| Milvus        | Open source          | HNSW, IVF, PQ      | On-premise / cloud |
| Weaviate      | Open source          | HNSW               | On-premise / cloud |
| Qdrant        | Open source / Rust   | HNSW               | On-premise / cloud |
| Chroma        | Ligera, prototipos   | HNSW               | Local / cloud      |
| pgvector      | Extensión PostgreSQL | IVFFlat, HNSW      | On-premise / cloud |
| Elasticsearch | Motor de búsqueda    | HNSW (plugin)      | On-premise / cloud |
| RedisVL       | Módulo Redis         | HNSW               | On-premise / cloud |

---

## 4️⃣ Flujo RAG (Retrieval Augmented Generation)

1. **Indexación:** dividir documentos en chunks → generar embeddings → almacenar en base vectorial.
2. **Consulta:** usuario hace pregunta.
3. **Embedding de consulta:** generar vector de la pregunta.
4. **Búsqueda de similitud:** recuperar k fragmentos más cercanos.
5. **Generación de contexto:** construir prompt con fragmentos + pregunta.
6. **LLM:** generar respuesta basada en contexto.

**Orquestadores populares:**

* LangChain
* LlamaIndex
* Haystack

---

## 5️⃣ Integración con SQL / NoSQL

* Metadatos en PostgreSQL / MongoDB, embeddings en pgvector o base vectorial → búsquedas híbridas.
* Estrategias de consistencia: actualización síncrona o CDC (Change Data Capture).

**Ejemplo SQL (pgvector + filtro):**

```sql
SELECT nombre, precio
FROM productos
WHERE precio > 50 AND categoria='electronica'
ORDER BY embedding <=> (SELECT embedding FROM productos WHERE id=123)
LIMIT 10;
```

---

## 6️⃣ Ejemplos de Uso

* **Búsqueda semántica en e-commerce:** "vestido rojo elegante" → resultados relevantes.
* **FAQ / chatbots:** documentos internos → respuestas contextuales.
* **Recomendaciones:** embeddings de películas, canciones, etc.
* **Detección de plagio:** comparar embeddings de textos.
* **Búsqueda multimodal:** CLIP, texto ↔ imagen.

---

## 7️⃣ Generación de Embeddings (Python)

```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('all-MiniLM-L6-v2')
oraciones = ["Gato en la alfombra", "Perro en el parque"]
embeddings = model.encode(oraciones)  # shape (2, 384)
```

---

## 8️⃣ Ejemplo RAG con LangChain + Chroma

```python
from langchain.embeddings import HuggingFaceEmbeddings
from langchain.vectorstores import Chroma
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA

embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
vectorstore = Chroma.from_documents(docs, embeddings)
qa = RetrievalQA.from_chain_type(llm=OpenAI(), retriever=vectorstore.as_retriever())
respuesta = qa.run("¿Cuál es el tema principal del documento?")
```

---

## 9️⃣ Despliegue

| Tipo       | Tecnologías / Ventajas                               | Desventajas                                        |                                             |
| ---------- | ---------------------------------------------------- | -------------------------------------------------- | ------------------------------------------- |
| On-Premise | Milvus, Weaviate, Qdrant, pgvector                   | Control total, seguridad                           | Escalado manual, mantenimiento              |
| Cloud      | Pinecone, Milvus Cloud, Weaviate Cloud, Qdrant Cloud | Escalabilidad automática, menor overhead operativo | Costos variables, dependencia del proveedor |

---

## 🔟 Glosario Rápido

* **Embedding:** vector que representa un objeto.
* **ANN:** búsqueda aproximada de vecinos.
* **HNSW / IVF / PQ:** algoritmos de indexación.
* **RAG:** recuperación + generación de texto.
* **LangChain / LlamaIndex:** frameworks para RAG y LLMs.
* **pgvector:** extensión vectorial para PostgreSQL.
* **Pinecone / Milvus / Weaviate / Qdrant:** bases vectoriales.

