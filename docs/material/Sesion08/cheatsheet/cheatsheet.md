---
layout: default
---

# Cheatsheet: Ingeniería de datos (captura)
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-08](../../../sesiones/sesion-08.md)

---

## Modos

| Modo | Cuándo |
| :--- | :--- |
| **Batch** | Informes diarios, cargas históricas |
| **Streaming** | Fraude, métricas en vivo |

---

## Checklist de ingesta

1. Autenticación / cuotas API  
2. Formato (JSON, CSV, Avro, Parquet)  
3. Esquema + evolución  
4. Dedup / idempotencia  

---

## Teoría

* [teoria8-bbdd.md](../teoria/teoria8-bbdd.md)

---

## Python (esqueleto requests)

```python
import requests
r = requests.get(url, timeout=30)
r.raise_for_status()
data = r.json()
```

---

## Puntos críticos

* **Scraping** sin cumplir términos de uso puede ser ilegal o bloqueante IP.
* Guardar **raw** antes de limpiar permite reprocesar.

> *“Captura estable = contratos de esquema + reintentos con backoff.”*
