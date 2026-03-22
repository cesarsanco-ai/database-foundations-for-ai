---
layout: default
---

# Cheatsheet: Big Data
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-10](../../../sesiones/sesion-10.md)

---

## 5 V (recordatorio)

| V | Pregunta |
| :--- | :--- |
| Volumen | ¿Cuántos TB/PB? |
| Velocidad | ¿Batch o near real-time? |
| Variedad | ¿JSON, logs, imágenes? |
| Veracidad | ¿Calidad y sesgos? |
| Valor | ¿Caso de negocio claro? |

---

## Formatos

| Formato | Nota |
| :--- | :--- |
| **Parquet** | Columnar, muy usado en lakes |
| **Avro** | Esquema evolutivo en streaming |
| **ORC** | Columnar Hive/Spark |

---

## Teoría

* [teoria10-bbdd.md](../teoria/teoria10-bbdd.md)

---

## Spark (idea)

```python
df = spark.read.parquet("s3://bucket/events/")
df.groupBy("day").count().write.mode("overwrite").parquet("out/")
```

---

## Puntos críticos

* **Shuffle** caro: minimizar `groupBy` innecesarios y reparticiones.
* Particionar por **fecha** o clave de negocio reduce scans.

> *“Big Data sin gobierno es big chaos.”*
