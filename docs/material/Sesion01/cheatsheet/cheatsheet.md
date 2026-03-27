---
layout: default
---

# Cheatsheet: Universo del dato
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-01](../../../sesiones/sesion-01.md)

---

## 1. Jerarquía: Dato → Información → Conocimiento

| Concepto | Definición | Ejemplo en BBDD |
| :--- | :--- | :--- |
| **Dato** | Hecho objetivo y crudo sin contexto. | `1010`, `"Pérez"`, `2025-03-15` |
| **Información** | Dato procesado y contextualizado. | Consulta SQL que une tablas para saber que `1010` es el ID del cliente `Juan Pérez`. |
| **Conocimiento** | Información aplicada para acción o predicción. | Detectar patrón de compra para lanzar una campaña de marketing. |

## 2. Clasificación de Datos por Estructura

| Tipo | Características | Ejemplo |
| :--- | :--- | :--- |
| **Estructurados** | Esquema rígido (filas y columnas). | Tablas SQL (Clientes, Pedidos). |
| **Semiestructurados** | Organización flexible con etiquetas. | JSON, XML, YAML. |
| **No estructurados** | Sin organización predefinida. | Texto libre, imágenes, logs, vídeo. |

## 3. Tipos de Datos y Codificación

### Tipos Elementales
- **Numéricos**: `INT`, `BIGINT`, `FLOAT`, `DECIMAL` (usar `DECIMAL` para finanzas).
- **Texto**: `CHAR` (fijo), `VARCHAR` (variable), `TEXT`.
- **Fecha/Hora**: `DATE`, `TIME`, `TIMESTAMP` (cuidado con husos horarios).
- **Binarios**: `BLOB` (imágenes, archivos).
- **Booleanos**: `BOOL`.

### Codificación de Caracteres
- **Recomendación universal**: **UTF-8**. Evita problemas de caracteres ilegibles (mojibake).
- *MySQL/PostgreSQL*: Usar `utf8mb4` (soporte completo para emojis y caracteres especiales).

## 4. Formatos de Almacenamiento

| Formato | Uso Principal | Ventajas | Desventajas |
| :--- | :--- | :--- | :--- |
| **CSV** | Intercambio simple. | Legible, soporte universal. | Sin esquema, ineficiente, lectura secuencial. |
| **JSON** | APIs, documentos NoSQL. | Flexible, datos anidados. | Verboso, lento para parsear. |
| **Avro** | Pipelines en tiempo real. | Esquema evolutivo, compresión. | No legible. |
| **Parquet** | Data Warehouses, Big Data. | **Columnar**: lee solo columnas necesarias, alta compresión. | No apto para escrituras registro a registro. |

> **Regla de oro para IA/Big Data**: Usar **Parquet** con compresión (Snappy, Zstandard). Reduce costos de almacenamiento y acelera lecturas drásticamente.

## 5. Metadata y Gobernanza

### Tipos de Metadata
- **Descriptiva**: Descubre e identifica (título, autor).
- **Estructural**: Describe formato y estructura (longitud de campo, tipo de dato).
- **Administrativa**: Información técnica (origen, permisos, fecha de creación).

### Conceptos Clave
- **Data Lineage (Linaje)**: Rastrear el origen y las transformaciones de un dato. Esencial para auditorías y depurar sesgos en IA.
- **Gobernanza**: Políticas de calidad, seguridad, privacidad (GDPR, CCPA). Define quién, cómo y cuándo se usan los datos.

## 6. Hardware y Jerarquía de Memoria


| Nivel                  | Latencia Aprox.   | Rol en BBDD                                                                                         |
| :--------------------- | :---------------- | :-------------------------------------------------------------------------------------------------- |
| **Registros CPU (L1)** | ~1 ns             | Memoria interna del CPU, ultrarrápida; se usa para operaciones aritméticas inmediatas.              |
| **RAM (Buffer Pool)**  | ~100 ns           | Memoria principal donde la BBDD mantiene datos y páginas en caché para acceso rápido.               |
| **NVMe / SSD**         | 20 µs - 0.2 ms    | Discos de estado sólido rápidos; almacenan datos persistentes con baja latencia.                    |
| **HDD**                | 4 - 10 ms         | Discos mecánicos más lentos; adecuados para almacenamiento económico pero no para alto rendimiento. |
| **SAN**                | +0.5 ms adicional | Red dedicada de almacenamiento que conecta múltiples servidores a discos compartidos.               |




## 7. Complejidad de Búsqueda (Índices)

| Método | Estructura | Complejidad | Uso Típico |
| :--- | :--- | :--- | :--- |
| **Table Scan** | Secuencial | **O(n)** | Sin índices o tablas muy pequeñas. |
| **B-Tree** | Árbol balanceado | **O(log n)** | Índices por defecto en SQL. Ideal para igualdad y rangos. |
| **Hash** | Tabla hash | **O(1)** promedio | Solo búsquedas de igualdad exacta (ej. cachés en memoria). |

### Impacto Práctico
- **Con índice** (`WHERE apellido = 'García'` en 1M filas): ~20 accesos.
- **Sin índice** (Table Scan): 1,000,000 accesos.
- **Costo de escritura**: Los índices aceleran `SELECT` pero ralentizan `INSERT/UPDATE/DELETE`.

## 8. Escala de Datos

| Unidad | Bytes | Contexto |
| :--- | :--- | :--- |
| **KB / MB** | 10³ / 10⁶ | Documentos, fotos. |
| **GB / TB** | 10⁹ / 10¹² | Películas, discos duros. |
| **PB / EB** | 10¹⁵ / 10¹⁸ | Grandes empresas, Internet global. |

*Ejemplo*: Un banco puede manejar varios **PB**; redes sociales ingieren **PB por día**.

## 9. Roles Clave en Datos

| Rol | Responsabilidad Principal |
| :--- | :--- |
| **DBA** | Operación: instalación, backups, seguridad, tuning de BBDD. |
| **Data Engineer** | Infraestructura: pipelines ETL/ELT, data lakes, orquestación (Spark, Airflow). |
| **Data Analyst** | Análisis: reportes, dashboards, responder preguntas de negocio. |
| **Data Scientist** | Modelado: IA/ML, predicciones. |
| **Data Steward** | Gobernanza: calidad, documentación, cumplimiento normativo. |

## 10. IA y Bases de Datos: El Paradigma RAG

### Problema Clásico
- **Ventana de Contexto**: Memoria a corto plazo del LLM. Aunque es grande (ej. 1M tokens), es finita y costosa.
- **Latencia**: Si la IA consulta la BBDD sin índices (Table Scan) → tiempo de respuesta inaceptable.

### Solución: Retrieval-Augmented Generation (RAG)
1.  **Embedding**: Convertir consulta en vector.
2.  **Búsqueda Vectorial**: Usar índices especializados (**HNSW**, **IVF**) para encontrar datos relevantes en tiempo sublineal.
3.  **Inyección**: Contexto recuperado + pregunta original → LLM.
4.  **Generación**: Respuesta fundamentada en datos actualizados.

### Riesgos Críticos (Alucinaciones)
- **Datos incorrectos/desactualizados** en la fuente.
- **Recuperación fallida**: Índice mal calibrado o timeout en consulta SQL.
- **Conclusión**: La velocidad y precisión de la base de datos son la primera línea de defensa contra la "alucinación" de la IA.