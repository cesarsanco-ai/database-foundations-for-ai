---
layout: default
---

# Fundamento: bases de datos vectoriales, similitud y RAG
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-12](../../../sesiones/sesion-12.md)

*(Alineado con la teoría de la Semana 12: [teoria12-bbdd.md](../teoria/teoria12-bbdd.md).)*

---

## 1. Embeddings

Vectores densos que representan texto/imagen/audio; “cercanía” ≈ similitud semántica según métrica elegida.

---

## 2. Búsqueda aproximada (ANN)

**HNSW, IVF, LSH** — equilibrio recall vs latencia vs memoria; parámetros `ef`, `nlist`, etc.

---

## 3. RAG

Recuperar pasajes relevantes (vector DB) + prompt al LLM; reduce alucinación si el contexto es fiel.

---

## 4. Operaciones

Inserción de vectores, filtros híbridos (metadata + similitud), reindexación al cambiar modelo de embedding.

---

## 5. Enlaces directos

* **Teoría completa:** [teoria12-bbdd.md](../teoria/teoria12-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
