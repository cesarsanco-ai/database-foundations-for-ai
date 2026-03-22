---
layout: default
---

# Fundamento: data warehousing, pipelines y procesamiento batch/streaming
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-09](../../../sesiones/sesion-09.md)

*(Alineado con la teoría de la Semana 9: [teoria9-bbdd.md](../teoria/teoria9-bbdd.md).)*

---

## 1. DW vs Lake vs Lakehouse

**DW:** SQL analítico, esquema fuerte. **Lake:** datos crudos baratos. **Lakehouse:** transacciones ACID sobre formatos abiertos (p. ej. Delta, Iceberg).

---

## 2. ETL vs ELT

**ETL:** transformar fuera del DW. **ELT:** cargar crudo y transformar *dentro* del motor analítico (típico en cloud).

---

## 3. Batch vs streaming

Ventanas temporales, **exactly-once** vs **at-least-once**, idempotencia en sinks.

---

## 4. Capas y gobernanza

Raw → staging → integración → datamarts; **metadatos**, catálogos, linaje.

---

## 5. Orquestación

Jobs, DAGs (Airflow, Dagster, etc.), SLAs y reintentos.

---

## 6. Enlaces directos

* **Teoría completa:** [teoria9-bbdd.md](../teoria/teoria9-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
