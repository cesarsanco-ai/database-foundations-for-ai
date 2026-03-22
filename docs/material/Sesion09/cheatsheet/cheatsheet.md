---
layout: default
---

# Cheatsheet: Pipelines de datos
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-09](../../../sesiones/sesion-09.md)

---

## ETL vs ELT

| Enfoque | Dónde transformas |
| :--- | :--- |
| **ETL** | Motor intermedio / cluster |
| **ELT** | Dentro del warehouse/lakehouse |

---

## Comparativa rápida

| Plataforma | Fortaleza típica |
| :--- | :--- |
| DW cloud | SQL masivo, BI |
| Data lake | Coste + variedad de formatos |
| Lakehouse | ACID + SQL sobre Parquet/Delta |

---

## Batch vs streaming

| Pregunta | Batch | Streaming |
| :--- | :--- | :--- |
| Latencia | Minutos–días | ms–s |
| Herramientas | Spark batch, cron | Flink, Kafka Streams |

---

## Teoría

* [teoria9-bbdd.md](../teoria/teoria9-bbdd.md)

---

## Puntos críticos

* Sin **metadatos** (quién, cuándo, versión del esquema) el lake se convierte en pantano.
* Coste = almacenamiento + **scan** + egress; optimizar particiones y formatos columnar.

> *“Un pipeline no termina en la tabla: termina en el contrato de datos con el consumidor.”*
