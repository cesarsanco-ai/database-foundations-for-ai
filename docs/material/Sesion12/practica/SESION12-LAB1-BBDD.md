# 📘 Laboratorio 7: Bases de Datos Vectoriales y RAG con MongoDB Atlas

## 🎯 Objetivo

Implementar una base de datos vectorial en MongoDB Atlas para el caso Aletza, permitiendo búsquedas semánticas en los mensajes y metadatos del asistente, y construir un pipeline RAG (Retrieval Augmented Generation) básico dentro del ecosistema de MongoDB.

Al finalizar, podrás:
- Crear un cluster con soporte vectorial en MongoDB Atlas
- Generar embeddings de texto usando modelos de IA
- Almacenar y consultar vectores en MongoDB
- Realizar búsquedas por similitud semántica
- Construir un sistema RAG simple para respuestas inteligentes

---

## 📋 Requisitos Previos

- Cuenta en MongoDB Atlas (gratuita)
- Cluster M0 creado (del laboratorio anterior)
- Datos de Aletza cargados en colecciones
- Conocimientos básicos de Python (para generación de embeddings)

---

## 🌐 Paso 1: Preparación del Entorno en MongoDB Atlas

### 1.1 Verificar que Atlas Search está Habilitado

MongoDB Atlas Search es el servicio que permite búsqueda vectorial en la capa gratuita.

1. Ir a **"Atlas Search"** en el panel lateral
2. Si es la primera vez, hacer clic en **"Create Search Index"**

### 1.2 Crear Índice de Búsqueda Vectorial

**Importante:** Los índices vectoriales en Atlas Search requieren que los campos vectoriales estén definidos.

1. Seleccionar la base de datos `aletza_db`
2. Seleccionar la colección `mensajes`
3. Hacer clic en **"Create Search Index"**
4. Seleccionar **"JSON Editor"** (no el visual editor)
5. Pegar la siguiente configuración:

```json
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "embedding": {
        "type": "knnVector",
        "dimensions": 384,
        "similarity": "cosine"
      },
      "contenido": {
        "type": "string",
        "analyzer": "lucene.spanish"
      },
      "metadata_ia.intencion": {
        "type": "string"
      },
      "metadata_ia.sentimiento": {
        "type": "string"
      },
      "modalidad": {
        "type": "string"
      }
    }
  }
}
```

6. Hacer clic en **"Create Search Index"**
7. Esperar a que el índice se cree (1-2 minutos)

---

## 🐍 Paso 2: Generación de Embeddings con Python

### 2.1 Configurar Entorno Python

**Nota:** Este paso requiere Python. Si no tienes Python instalado, puedes usar **Google Colab** o un entorno online.

```bash
# Instalar dependencias
pip install pymongo sentence-transformers python-dotenv
```

### 2.2 Obtener la Cadena de Conexión de MongoDB

1. En Atlas, hacer clic en **"Connect"**
2. Seleccionar **"Connect your application"**
3. Copiar la cadena de conexión
4. Reemplazar `<password>` con tu contraseña

### 2.3 Generar y Almacenar Embeddings

```python
import pymongo
from sentence_transformers import SentenceTransformer
import time

# Conexión a MongoDB Atlas
MONGO_URI = "mongodb+srv://aletza_user:<password>@aletza-cluster.xxxxx.mongodb.net/"
client = pymongo.MongoClient(MONGO_URI)
db = client["aletza_db"]
mensajes_collection = db["mensajes"]

# Cargar modelo de embeddings (384 dimensiones)
model = SentenceTransformer('all-MiniLM-L6-v2')
print("Modelo cargado correctamente")

# Obtener mensajes que no tienen embedding
mensajes = list(mensajes_collection.find({
    "embedding": {"$exists": False},
    "contenido": {"$exists": True, "$ne": None}
}).limit(50))

print(f"Generando embeddings para {len(mensajes)} mensajes...")

# Generar y actualizar embeddings
for mensaje in mensajes:
    contenido = mensaje["contenido"]
    if contenido:
        # Generar embedding
        embedding = model.encode(contenido).tolist()
        
        # Actualizar documento
        mensajes_collection.update_one(
            {"_id": mensaje["_id"]},
            {"$set": {"embedding": embedding}}
        )
        print(f"✓ Embedding generado para mensaje ID: {mensaje['_id']}")
        time.sleep(0.1)  # Pequeña pausa para no saturar

print("Proceso completado!")
client.close()
```

---

## 🔍 Paso 3: Búsqueda Vectorial en MongoDB Atlas

### 3.1 Búsqueda por Similitud Semántica (Desde Atlas)

En MongoDB Atlas, podemos realizar búsquedas vectoriales usando el pipeline de agregación con la etapa `$search`.

**Para ejecutar desde Atlas:**
1. Ir a la colección `mensajes`
2. Ir a la pestaña **"Aggregations"**
3. Agregar una etapa con `$search` (Stage: SEARCH)
4. Configurar la búsqueda vectorial

**Pipeline de agregación para búsqueda semántica:**

```json
[
    {
        "$search": {
            "index": "default",
            "knnBeta": {
                "vector": [0.1, 0.2, 0.3, ...],  // Vector de consulta (384 dimensiones)
                "path": "embedding",
                "k": 5
            }
        }
    },
    {
        "$project": {
            "contenido": 1,
            "modalidad": 1,
            "metadata_ia": 1,
            "score": { "$meta": "searchScore" }
        }
    }
]
```

### 3.2 Búsqueda Vectorial con Filtros

**Ejemplo:** Buscar mensajes similares a "problemas con la cuenta" pero solo los que tienen sentimiento negativo

```json
[
    {
        "$search": {
            "index": "default",
            "compound": {
                "must": [
                    {
                        "knnBeta": {
                            "vector": [0.1, 0.2, 0.3, ...],
                            "path": "embedding",
                            "k": 10
                        }
                    }
                ],
                "filter": [
                    {
                        "text": {
                            "path": "metadata_ia.sentimiento",
                            "query": "negativo"
                        }
                    }
                ]
            }
        }
    },
    {
        "$project": {
            "contenido": 1,
            "metadata_ia": 1,
            "score": { "$meta": "searchScore" }
        }
    }
]
```

---

## 🐍 Paso 4: Búsqueda Vectorial con Python

### 4.1 Función de Búsqueda Semántica

```python
import pymongo
from sentence_transformers import SentenceTransformer
import numpy as np

def buscar_similares(pregunta, coleccion, limite=5):
    """
    Busca mensajes similares a una pregunta usando embeddings.
    """
    # Generar embedding de la pregunta
    model = SentenceTransformer('all-MiniLM-L6-v2')
    vector_pregunta = model.encode(pregunta).tolist()
    
    # Pipeline de agregación con búsqueda vectorial
    pipeline = [
        {
            "$search": {
                "index": "default",
                "knnBeta": {
                    "vector": vector_pregunta,
                    "path": "embedding",
                    "k": limite
                }
            }
        },
        {
            "$project": {
                "contenido": 1,
                "modalidad": 1,
                "metadata_ia": 1,
                "score": { "$meta": "searchScore" }
            }
        }
    ]
    
    # Ejecutar búsqueda
    resultados = list(coleccion.aggregate(pipeline))
    return resultados

# Ejemplo de uso
MONGO_URI = "mongodb+srv://aletza_user:<password>@aletza-cluster.xxxxx.mongodb.net/"
client = pymongo.MongoClient(MONGO_URI)
db = client["aletza_db"]
mensajes = db["mensajes"]

# Buscar mensajes similares a una consulta
consulta = "¿Cómo soluciono problemas con mi cuenta?"
resultados = buscar_similares(consulta, mensajes, limite=3)

print(f"\nPregunta: {consulta}")
print("Mensajes similares encontrados:\n")
for i, r in enumerate(resultados, 1):
    print(f"{i}. {r['contenido']}")
    print(f"   Score: {r['score']:.4f}\n")
```

### 4.2 Búsqueda Híbrida (Vectorial + Filtros)

```python
def buscar_similares_con_filtro(pregunta, coleccion, filtros=None, limite=5):
    """
    Busca mensajes similares aplicando filtros adicionales.
    """
    model = SentenceTransformer('all-MiniLM-L6-v2')
    vector_pregunta = model.encode(pregunta).tolist()
    
    # Construir pipeline con filtros
    if filtros:
        # Convertir filtros a formato Atlas Search
        condiciones = []
        for campo, valor in filtros.items():
            if '.' in campo:
                # Campo anidado (ej: metadata_ia.sentimiento)
                condiciones.append({
                    "text": {
                        "path": campo,
                        "query": valor
                    }
                })
            else:
                condiciones.append({
                    "equals": {
                        "path": campo,
                        "value": valor
                    }
                })
        
        pipeline = [
            {
                "$search": {
                    "index": "default",
                    "compound": {
                        "must": [
                            {
                                "knnBeta": {
                                    "vector": vector_pregunta,
                                    "path": "embedding",
                                    "k": limite * 2
                                }
                            }
                        ],
                        "filter": condiciones
                    }
                }
            },
            {
                "$limit": limite
            },
            {
                "$project": {
                    "contenido": 1,
                    "modalidad": 1,
                    "metadata_ia": 1,
                    "score": { "$meta": "searchScore" }
                }
            }
        ]
    else:
        pipeline = [
            {
                "$search": {
                    "index": "default",
                    "knnBeta": {
                        "vector": vector_pregunta,
                        "path": "embedding",
                        "k": limite
                    }
                }
            },
            {
                "$project": {
                    "contenido": 1,
                    "modalidad": 1,
                    "metadata_ia": 1,
                    "score": { "$meta": "searchScore" }
                }
            }
        ]
    
    return list(coleccion.aggregate(pipeline))

# Ejemplo: buscar soluciones técnicas con sentimiento negativo
filtros = {
    "metadata_ia.intencion": "soporte",
    "metadata_ia.sentimiento": "negativo"
}

resultados = buscar_similares_con_filtro(
    "error al iniciar sesión", 
    mensajes, 
    filtros=filtros,
    limite=3
)

print("Resultados filtrados:")
for r in resultados:
    print(f"- {r['contenido']} (score: {r['score']:.4f})")
```

---

## 🤖 Paso 5: Construcción de un Sistema RAG Básico

### 5.1 Arquitectura RAG en MongoDB

```
Usuario → Pregunta → Embedding → Búsqueda Vectorial → Contexto → LLM → Respuesta
                                    ↓
                            MongoDB Atlas
                            (mensajes con embeddings)
```

### 5.2 Implementación del RAG

```python
import pymongo
from sentence_transformers import SentenceTransformer
import requests
import json

class RAGSystem:
    def __init__(self, mongo_uri, db_name, collection_name):
        self.client = pymongo.MongoClient(mongo_uri)
        self.db = self.client[db_name]
        self.collection = self.db[collection_name]
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        
        # Opcional: Configurar LLM (OpenAI, Ollama, etc.)
        self.llm_url = None  # Configurar si se usa una API
    
    def buscar_contexto(self, pregunta, limite=5):
        """Busca mensajes relevantes para la pregunta"""
        vector = self.model.encode(pregunta).tolist()
        
        pipeline = [
            {
                "$search": {
                    "index": "default",
                    "knnBeta": {
                        "vector": vector,
                        "path": "embedding",
                        "k": limite
                    }
                }
            },
            {
                "$project": {
                    "contenido": 1,
                    "metadata_ia": 1,
                    "score": { "$meta": "searchScore" }
                }
            }
        ]
        
        return list(self.collection.aggregate(pipeline))
    
    def construir_prompt(self, pregunta, contexto):
        """Construye el prompt para el LLM"""
        prompt = f"""Eres Aletza, un asistente virtual inteligente. 
Responde la pregunta del usuario basándote en el contexto proporcionado.

Contexto (mensajes anteriores similares):
"""
        for i, c in enumerate(contexto, 1):
            prompt += f"{i}. {c['contenido']}\n"
        
        prompt += f"\nPregunta del usuario: {pregunta}\n"
        prompt += "\nRespuesta (basada en el contexto):"
        
        return prompt
    
    def responder(self, pregunta, usar_llm=False):
        """Genera una respuesta usando el contexto recuperado"""
        contexto = self.buscar_contexto(pregunta)
        
        if not contexto:
            return "Lo siento, no encontré información relevante para responder tu pregunta."
        
        if usar_llm and self.llm_url:
            # Usar LLM externo
            prompt = self.construir_prompt(pregunta, contexto)
            # Llamada a API (ejemplo con Ollama)
            response = requests.post(
                self.llm_url,
                json={"prompt": prompt, "model": "llama2"}
            )
            return response.json().get("response", "Error en el LLM")
        else:
            # Respuesta simple usando el contexto más relevante
            mejor_contexto = contexto[0]
            return f"Según conversaciones anteriores similares:\n{mejor_contexto['contenido']}\n\n¿Te ayuda esta información?"

# Crear instancia del sistema RAG
rag = RAGSystem(
    mongo_uri="mongodb+srv://aletza_user:<password>@aletza-cluster.xxxxx.mongodb.net/",
    db_name="aletza_db",
    collection_name="mensajes"
)

# Probar el sistema
preguntas = [
    "¿Cómo puedo solucionar problemas con mi cuenta?",
    "¿Qué funciones tiene el asistente?",
    "¿Cómo configuro mi perfil de voz?"
]

for pregunta in preguntas:
    print(f"\n{'='*50}")
    print(f"Pregunta: {pregunta}")
    print(f"{'='*50}")
    respuesta = rag.responder(pregunta, usar_llm=False)
    print(f"Respuesta: {respuesta}\n")
```

---

## 📊 Paso 6: Visualización de Embeddings en MongoDB

### 6.1 Verificar que los embeddings se generaron

```javascript
// En MongoDB Atlas, en la colección mensajes
// Filtrar mensajes que tienen embedding
{ "embedding": { "$exists": true } }
```

### 6.2 Ver estadísticas de vectores

```javascript
// Contar mensajes con embedding
db.mensajes.countDocuments({ "embedding": { "$exists": true } })

// Ver un ejemplo de embedding (primeros 5 valores)
db.mensajes.findOne({ "embedding": { "$exists": true } }, { "embedding": { "$slice": 5 } })
```

---

## 🧪 Paso 7: Ejercicios Prácticos

### Ejercicio 1: Generar Embeddings para Todos los Mensajes

```python
# Script para generar embeddings de todos los mensajes sin embedding
def generar_todos_los_embeddings():
    mensajes_sin_embedding = list(mensajes_collection.find({
        "embedding": {"$exists": False},
        "contenido": {"$exists": True, "$ne": None}
    }))
    
    for mensaje in mensajes_sin_embedding:
        embedding = model.encode(mensaje["contenido"]).tolist()
        mensajes_collection.update_one(
            {"_id": mensaje["_id"]},
            {"$set": {"embedding": embedding}}
        )
        print(f"✓ ID: {mensaje['_id']}")
    
    print(f"Total: {len(mensajes_sin_embedding)} embeddings generados")

generar_todos_los_embeddings()
```

### Ejercicio 2: Búsqueda Semántica por Intención

```python
# Encontrar mensajes similares a una intención específica
def buscar_por_intencion(intencion, pregunta):
    filtros = {"metadata_ia.intencion": intencion}
    resultados = buscar_similares_con_filtro(pregunta, mensajes, filtros, limite=3)
    
    print(f"\nBuscando '{intencion}' similares a: '{pregunta}'")
    for r in resultados:
        print(f"- {r['contenido']}")
    
    return resultados

# Ejemplo
buscar_por_intencion("soporte", "no puedo iniciar sesión")
```

### Ejercicio 3: Crear un Índice de Embeddings para Perfiles de Voz

```python
# Crear embeddings para perfiles de voz (vector_voz ya existe como array)
# Pero podemos generar embeddings de texto descriptivo
perfiles = db["perfiles_voz"]

# Agregar campo descriptivo basado en estado
for perfil in perfiles.find({}):
    descripcion = f"Perfil de voz {perfil['estado']} con {perfil['muestras_tomadas']} muestras"
    
    # Generar embedding de la descripción
    embedding = model.encode(descripcion).tolist()
    
    perfiles.update_one(
        {"_id": perfil["_id"]},
        {"$set": {"descripcion_embedding": embedding, "descripcion_texto": descripcion}}
    )
    print(f"✓ Perfil ID: {perfil['_id']}")
```

### Ejercicio 4: Sistema de Recomendación de Mensajes

```python
def recomendar_mensajes_similares(mensaje_id, limite=3):
    """Recomienda mensajes similares a un mensaje dado"""
    # Obtener el mensaje original
    mensaje_original = mensajes_collection.find_one({"_id": mensaje_id})
    
    if not mensaje_original or "embedding" not in mensaje_original:
        return []
    
    # Buscar mensajes similares (excluyendo el original)
    pipeline = [
        {
            "$search": {
                "index": "default",
                "knnBeta": {
                    "vector": mensaje_original["embedding"],
                    "path": "embedding",
                    "k": limite + 1
                }
            }
        },
        {
            "$match": {
                "_id": {"$ne": mensaje_id}
            }
        },
        {
            "$limit": limite
        },
        {
            "$project": {
                "contenido": 1,
                "modalidad": 1,
                "score": { "$meta": "searchScore" }
            }
        }
    ]
    
    return list(mensajes_collection.aggregate(pipeline))

# Ejemplo: recomendar mensajes similares al mensaje con ID 1
recomendaciones = recomendar_mensajes_similares(1)
print("Mensajes similares recomendados:")
for r in recomendaciones:
    print(f"- {r['contenido']} (score: {r['score']:.4f})")
```

---

## 📈 Paso 8: Optimización y Mejores Prácticas

### 8.1 Configuración de Índices Vectoriales

```json
// Índice optimizado para búsqueda con filtros
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "embedding": {
        "type": "knnVector",
        "dimensions": 384,
        "similarity": "cosine"
      },
      "metadata_ia.intencion": {
        "type": "string"
      },
      "metadata_ia.sentimiento": {
        "type": "string"
      },
      "modalidad": {
        "type": "string"
      },
      "fecha": {
        "type": "date"
      }
    }
  }
}
```

### 8.2 Recomendaciones de Dimensionalidad

| Modelo | Dimensiones | Uso Recomendado |
|--------|-------------|-----------------|
| all-MiniLM-L6-v2 | 384 | Texto corto, bueno para mensajes |
| text-embedding-ada-002 | 1536 | Texto largo, más preciso |
| all-mpnet-base-v2 | 768 | Balance entre calidad y tamaño |

### 8.3 Estrategias para Mejorar RAG

1. **Chunking adecuado**: Dividir textos largos en fragmentos de 200-500 palabras
2. **Metadatos enriquecidos**: Incluir fecha, usuario, intención en los filtros
3. **Re-ranking**: Refinar resultados después de la búsqueda vectorial
4. **Cache de consultas frecuentes**: Almacenar resultados de búsquedas comunes

---

## ✅ Verificación Final

```javascript
// Verificar que los índices están creados
// En Atlas, ir a "Atlas Search" y verificar el índice

// Verificar cantidad de mensajes con embedding
db.mensajes.countDocuments({ "embedding": { "$exists": true } })

// Probar búsqueda vectorial simple
// En la interfaz de Atlas, usar el pipeline de agregación con $search
```

---

## 📝 Resumen de Conceptos Aplicados

| Concepto | Implementación en Aletza |
|----------|-------------------------|
| **Embeddings** | Mensajes convertidos a vectores de 384 dimensiones con all-MiniLM-L6-v2 |
| **Índice Vectorial** | Atlas Search con tipo `knnVector` y similitud coseno |
| **Búsqueda Semántica** | Pipeline `$search` con `knnBeta` |
| **Búsqueda con Filtros** | `compound` con `must` y `filter` |
| **RAG** | Contexto recuperado + respuesta generada (LLM opcional) |
| **Recomendación** | Mensajes similares basados en embeddings |

---

## 🎯 Conclusión

Hemos implementado un sistema de búsqueda semántica y RAG en MongoDB Atlas para el asistente Aletza:

1. **Generamos embeddings** de los mensajes usando Sentence Transformers
2. **Almacenamos vectores** en MongoDB junto con los documentos originales
3. **Creamos índices vectoriales** con Atlas Search para búsqueda eficiente
4. **Realizamos búsquedas semánticas** con filtros por intención, sentimiento y modalidad
5. **Construimos un sistema RAG básico** que recupera contexto relevante
6. **Implementamos recomendaciones** basadas en similitud semántica

Esta arquitectura permite al asistente Aletza entender el significado de las preguntas de los usuarios, no solo palabras clave, mejorando significativamente la calidad de las respuestas.