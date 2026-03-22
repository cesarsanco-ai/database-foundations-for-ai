---
layout: default
---

# Cheatsheet: Bases de datos vectoriales
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-12](../../../sesiones/sesion-12.md)

---

## Métricas

| Métrica | Cuándo |
| :--- | :--- |
| **Coseno** | Embeddings L2-normalizados |
| **L2** | Distancia euclídea en espacio de embedding |
| **IP** | Producto interno (vectores normalizados ~ coseno) |

---

## Pipeline RAG (macro)

1. Chunking de documentos  
2. Embedding con modelo fijo  
3. `upsert` en vector store  
4. Query → top-k → contexto → LLM  

---

## Teoría

* [teoria12-bbdd.md](../teoria/teoria12-bbdd.md)

---

## Puntos críticos

* Mismo **modelo** de embedding en índice y en consulta.
* **Chunk size** y solapamiento afectan recall/precisión.

> *“ANN es probabilístico: mide recall@k en tu dominio.”*
