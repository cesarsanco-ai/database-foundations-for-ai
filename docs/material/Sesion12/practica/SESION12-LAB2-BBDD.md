# 🔬 Laboratorio 2: Debugging de Recuperación en RAG (El Caso "Playlist")

Este laboratorio documenta el fallo semántico donde un documento relevante queda fuera del Top-K y cómo aplicar técnicas de ingeniería de recuperación para solucionarlo.

---

## 🛠️ Fase 1: Preparación del Escenario (Setup)

### 1.1 El Corpus "Ruidoso"
Para replicar el fallo, necesitamos un corpus donde existan temas que "distraigan" al modelo de los temas musicales.
* **Dataset**: 20 mensajes.
* **Documento Objetivo (ID 7)**: `"Reproduce mi playlist favorita"` (Remitente 2, Intención: Música).
* **Documentos "Ruido"**: 15 mensajes sobre soporte técnico, clima y horarios (ej. *"¿Cuál es el horario?"*, *"No carga mi cuenta"*, *"Soporte técnico por favor"*).

### 1.2 Configuración del Índice en MongoDB Atlas
Crear un índice de tipo `vectorSearch` con las siguientes especificaciones:
```json
{
  "fields": [
    { "type": "vector", "path": "embedding", "numDimensions": 384, "similarity": "cosine" },
    { "type": "filter", "path": "metadata_ia.remitente_id" }
  ]
}
```

---

## 📝 Fase 2: Ejercicios de Diagnóstico

### Ejercicio 1: El Fallo del Top-K
**Instrucción**: Lanzar la query *"¿Algún usuario habló de playlist?"* con $K=3$.
* **Resultado esperado**: La respuesta será "No hay información".
* **Tarea**: Extraer los `scores` de los 10 primeros resultados. Identificar en qué posición ($rank$) quedó el Documento ID 7.



### Ejercicio 2: Análisis de Similitud Visual
**Instrucción**: Comparar el vector de la Query con el Vector del Doc ID 7 vs el Vector del Doc de Soporte Técnico.
* **Pregunta de reflexión**: ¿Por qué *"soporte técnico"* tuvo un score más alto que *"playlist"* ante la duda del usuario? (Pista: Densidad de términos comunes).

---

## 🚀 Fase 3: Estrategias de Mejora (Ajuste del Sistema)

### Estrategia A: Query Rewriting (HyDE - Hypothetical Document Embeddings)
En lugar de buscar con la pregunta del usuario, pedirle al LLM (Ollama) que genere una **respuesta hipotética** y buscar con esa respuesta.
* **Instrucción**: Prompt a Ollama: *"Escribe cómo se vería un mensaje de un usuario pidiendo una playlist"*.
* **Acción**: Usar ese texto generado para hacer el `vectorSearch`.



### Estrategia B: Inyección de Metadatos (Metadata Enrichment)
**Instrucción**: Modificar el script de generación de embeddings para que el texto de entrada sea una fusión.
* **Antes**: `"Reproduce mi playlist favorita"`
* **Ahora**: `"Mensaje de música del usuario 2: Reproduce mi playlist favorita [Intención: música]"`
* **Tarea**: Re-generar embeddings y medir si el Documento ID 7 sube en el ranking con $K=3$.

### Estrategia C: Re-ranking con Cross-Encoder
**Instrucción**: Configurar un flujo de dos pasos.
1. Recuperar $K=20$ de MongoDB.
2. Usar un modelo pequeño (ej. `cross-encoder/ms-marco-MiniLM-L-6-v2`) para re-evaluar esos 20 y seleccionar el Top-1 real.



---

## 📊 Fase 4: Medición y Métricas

Para validar la mejora, el alumno debe completar la siguiente tabla:

| Estrategia | Rank del Doc ID 7 | Score de Similitud | ¿Respondió la IA? |
| :--- | :--- | :--- | :--- |
| **Base (K=3)** | > 15 | 0.72 | No |
| **Metadata Injected** | ? | ? | ? |
| **HyDE Query** | ? | ? | ? |
| **Re-ranking (K=20)** | 1 | 0.92 (re-score) | Sí |

---

## 📖 Fase 5: Documentación del "Failure Mode"

**Consigna final**: Redactar un reporte de una página que explique:
1.  **El Síntoma**: La IA dice "no sé" a pesar de tener el dato.
2.  **La Causa Raíz**: Mismatch semántico y bajo $K$ ante un corpus con alta densidad de temas no relacionados.
3.  **La Solución Óptima**: Por qué el Re-ranking es la solución más escalable frente al aumento infinito de $K$.

---

### 🎯 Próximo paso para tu IA
**¿Quieres que genere ahora mismo el código Python listo para ejecutar la "Estrategia B" (Inyección de Metadatos)?** Es la forma más rápida de ver cómo el documento de la playlist sube del puesto 18 al puesto 1.






