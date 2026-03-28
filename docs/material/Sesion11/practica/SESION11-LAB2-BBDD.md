# 📘 Laboratorio 2: MongoDB Atlas 

## 🔍 Paso 4: Consultas Básicas en MongoDB Atlas

### 4.1 Cómo Realizar Consultas

En MongoDB Atlas, hay dos formas principales de consultar:

1. **Usando el campo "Filter"** (para consultas simples)
2. **Usando la pestaña "Aggregations"** (para consultas complejas con pipelines)

### 4.2 Filtrar Documentos (WHERE en SQL)

**Para usar el filtro:**
1. Seleccionar la colección
2. En el campo **"Filter"** (arriba de los documentos)
3. Pegar el JSON de filtro
4. Presionar **"Apply"**

| SQL | MongoDB Filter |
|-----|----------------|
| `SELECT * FROM usuarios WHERE pais = 'Perú'` | `{ "pais": "Perú" }` |
| `SELECT * FROM usuarios WHERE _id > 5` | `{ "_id": { "$gt": 5 } }` |
| `SELECT * FROM usuarios WHERE pais = 'Perú' AND ciudad = 'Lima'` | `{ "pais": "Perú", "ciudad": "Lima" }` |

**Ejemplos para probar en el campo Filter:**

```json
// Usuarios de Perú
{ "pais": "Perú" }

// Usuarios con ID mayor a 5
{ "_id": { "$gt": 5 } }

// Usuarios de Perú y Lima
{ "pais": "Perú", "ciudad": "Lima" }

// Mensajes con sentimiento positivo
{ "metadata_ia.sentimiento": "positivo" }

// Mensajes de modalidad audio
{ "modalidad": "audio" }
```

### 4.3 Operadores de Comparación

| SQL | MongoDB Filter |
|-----|----------------|
| `=` | `{ campo: valor }` |
| `!=` | `{ campo: { "$ne": valor } }` |
| `>` | `{ campo: { "$gt": valor } }` |
| `>=` | `{ campo: { "$gte": valor } }` |
| `<` | `{ campo: { "$lt": valor } }` |
| `<=` | `{ campo: { "$lte": valor } }` |

**Ejemplos:**
```json
// _id mayor que 5
{ "_id": { "$gt": 5 } }

```

### 4.4 Operadores Lógicos

| SQL | MongoDB Filter |
|-----|----------------|
| `AND` | `{ $and: [ {cond1}, {cond2} ] }` o simplemente `{ cond1, cond2 }` |
| `OR` | `{ $or: [ {cond1}, {cond2} ] }` |

**Ejemplos:**
```json
// AND implícito (ambas condiciones)
{ "pais": "Perú", "ciudad": "Lima" }

// OR explícito
{ "$or": [ { "pais": "Perú" }, { "pais": "Argentina" } ] }

// Combinación AND + OR
{
    "$and": [
        { "activo": true },
        { "$or": [ { "pais": "Perú" }, { "pais": "Argentina" } ] }
    ]
}
```

### 4.5 LIKE (Búsqueda de Patrones)

| SQL | MongoDB Filter |
|-----|----------------|
| `LIKE '%texto%'` | `{ campo: { "$regex": "texto", "$options": "i" } }` |

**Ejemplos:**
```json
// Usuarios con email que contiene "gmail"
{ "email": { "$regex": "gmail", "$options": "i" } }

// Mensajes que contienen "clima"
{ "contenido": { "$regex": "clima", "$options": "i" } }
```

### 4.6 BETWEEN (Rangos)

**Ejemplo:** Mensajes entre dos fechas
```json
{
    "enviado_en": {
        "$gte": { "$date": "2024-01-15T00:00:00Z" },
        "$lte": { "$date": "2024-01-16T23:59:59Z" }
    }
}
```

### 4.7 IN (Múltiples Valores)

**Ejemplo:** Usuarios de Perú, México y Argentina
```json
{ "pais": { "$in": ["Perú", "México", "Argentina"] } }
```

---

## 📊 Paso 5: Agregaciones (Equivalente a GROUP BY y JOINs)

### 5.1 Cómo Usar el Pipeline de Agregación

1. Seleccionar una colección (ej: `mensajes`)
2. Hacer clic en la pestaña **"Aggregations"**
3. Hacer clic en **"Add Stage"**
4. Seleccionar el tipo de etapa en el menú desplegable (ej: `$match`, `$group`, `$lookup`)
5. Escribir el JSON de la etapa
6. Hacer clic en **"Apply"** para ver resultados

### 5.2 Etapa `$match` (Filtrar) - STAGE: MATCH

**Equivalente a `WHERE` en SQL.**

**Seleccionar en el menú:** `$match`

**Ejemplo 1:** Filtrar mensajes con sentimiento positivo

```json
{
  "metadata_ia.sentimiento": "positivo"
}
```

**Ejemplo 2:** Filtrar mensajes de un usuario específico

```json
{
  "remitente_id": 1
}
```

### 5.3 Etapa `$group` (Agrupar) - STAGE: GROUP

**Equivalente a `GROUP BY` en SQL.**

**Seleccionar en el menú:** `$group`

**Ejemplo 1:** Contar mensajes por modalidad

```json
{
        "_id": "$modalidad",
        "total": { "$sum": 1 }
}
```


**Ejemplo 2:** Promedio de tiempo de ejecución por módulo

```json
{
        "_id": "$modulo",
        "tiempo_promedio": { "$avg": "$tiempo_ejecucion_ms" },
        "total": { "$sum": 1 }
}
```

**Ejemplo 3:** Conteo de usuarios por país

```json
{
        "_id": "$pais",
        "total_usuarios": { "$sum": 1 }
}
```

### 5.4 Etapa `$project` (Seleccionar/Crear campos) - STAGE: PROJECT

**Equivalente a `SELECT` con alias en SQL.**

**Seleccionar en el menú:** `$project`

**Ejemplo:** Crear campos de año y mes

```json
{
        "contenido": 1,
        "modalidad": 1,
        "anio": { "$year": "$enviado_en" },
        "mes": { "$month": "$enviado_en" }
}
```

### 5.5 Etapa `$sort` (Ordenar) - STAGE: SORT

**Equivalente a `ORDER BY` en SQL.**

**Seleccionar en el menú:** `$sort`

**Ejemplo:** Ordenar por fecha descendente

```json
{
        "enviado_en": -1
}
```

### 5.6 Etapa `$limit` (Limitar) - STAGE: LIMIT

**Equivalente a `LIMIT` en SQL.**

**Seleccionar en el menú:** `$limit`

**Ejemplo:** Mostrar solo 5 resultados

```json
5
```

### 5.7 Etapa `$lookup` (JOIN) - STAGE: LOOKUP

**Equivalente a `LEFT JOIN` en SQL.**

**Seleccionar en el menú:** `$lookup`

**Ejemplo:** Unir mensajes con usuarios

```json
{
        "from": "usuarios",
        "localField": "remitente_id",
        "foreignField": "_id",
        "as": "usuario_info"
}
```

**Después de $lookup, usar $unwind para expandir el array:**

**Seleccionar en el menú:** `$unwind`

```json
"$usuario_info"
```

### 5.8 Pipeline Completo: Mensajes por Usuario

**Paso a paso para construir el pipeline:**

**Stage 1 - LOOKUP:** Unir con usuarios
```json
{
        "from": "usuarios",
        "localField": "remitente_id",
        "foreignField": "_id",
        "as": "usuario"
}
```


**Stage 3 - GROUP:** Agrupar por usuario
```json
{
        "_id": "$usuario.username",
        "total_mensajes": { "$sum": 1 }
}

```

**Stage 4 - SORT:** Ordenar descendente
```json
{
        "total_mensajes": -1
}
```

### 5.9 Pipeline: Resumen Diario

**Stage 1 - GROUP:** Agrupar por fecha en usuarios
```json
{
        "_id": { "$dateToString": { "format": "%Y-%m-%d", "date": "$enviado_en" } },
        "total_mensajes": { "$sum": 1 },
        "usuarios_unicos": { "$addToSet": "$remitente_id" }
}
```

**Stage 2 - PROJECT:** Calcular usuarios activos
```json
{
        "fecha": "$_id",
        "total_mensajes": 1,
        "usuarios_activos": { "$size": "$usuarios_unicos" }
}


```

**Stage 3 - SORT:** Ordenar por fecha en usuarios
```json
{
        "fecha": 1
}


```

### 5.10 Pipeline: Top 3 Usuarios por Mensajes

**Stage 1 - GROUP:** Contar mensajes por usuario
```json
{
        "_id": "$remitente_id",
        "total": { "$sum": 1 }
}
```

**Stage 2 - SORT:** Ordenar descendente
```json
{
        "total": -1
}
```

**Stage 3 - LIMIT:** Top 3
```json
3
```

**Stage 4 - LOOKUP:** Obtener información del usuario
```json
{
        "from": "usuarios",
        "localField": "_id",
        "foreignField": "_id",
        "as": "usuario"
}
```

**Stage 5 - UNWIND:** Expandir
```json
"$usuario"
```

**Stage 6 - PROJECT:** Formatear resultado
```json
{
        "username": "$usuario.username",
        "total_mensajes": "$total"
}
```

---

## 🧪 Paso 6: Ejercicios Prácticos en MongoDB Atlas

### Ejercicio 1: Insertar Nuevo Documento

1. Ir a colección `usuarios`
2. Hacer clic en **"Insert Document"**
3. Seleccionar **"Insert Document"** (individual)
4. Insertar:

```json
{
    "_id": 11,
    "username": "nuevo_usuario",
    "email": "nuevo@test.com",
    "pais": "Brasil",
    "ciudad": "São Paulo",
    "activo": true,
    "creado_en": { "$date": "2025-03-27T10:00:00Z" }
}
```


---

## 🎯 Conclusión

MongoDB Atlas proporciona una interfaz web amigable para trabajar con datos NoSQL. Hemos:

1. **Creado un cluster gratuito** en MongoDB Atlas
2. **Insertado datos** del caso Aletza en formato documento JSON
3. **Realizado consultas** equivalentes a SQL usando el campo Filter
4. **Creado pipelines de agregación** con etapas `$match`, `$group`, `$lookup`, `$project`, `$sort`, `$limit`
5. **Optimizado consultas** con índices simples, compuestos y de texto

La flexibilidad de MongoDB permite almacenar datos semiestructurados (como `metadata_ia`) sin necesidad de definir un esquema fijo, lo que facilita la evolución del modelo de datos del asistente Aletza.