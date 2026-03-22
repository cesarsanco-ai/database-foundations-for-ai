---
layout: default
---

# Cheatsheet: Arquitecturas avanzadas y resiliencia
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-14](../../../sesiones/sesion-14.md)

---

## Comandos peligrosos

| Comando | Riesgo |
| :--- | :--- |
| `DROP DATABASE` | Irreversible sin backup |
| `TRUNCATE` | Rápido; a menudo sin undo por fila |
| `DELETE` sin `WHERE` | Borrado masivo |

---

## Estrategia de backup

1. Frecuencia acorde a RPO  
2. Off-site / otra región  
3. **Restore drill** trimestral  

---

## Híbrido SQL + NoSQL + Vector

| Capa | Rol |
| :--- | :--- |
| SQL | Verdad transaccional |
| NoSQL | Escala operacional / flexible |
| Vector | Búsqueda semántica / RAG |

---

## Teoría

* [teoria14-bbdd.md](../teoria/teoria14-bbdd.md)

---

## Puntos críticos

* **CDC** (Debezium, etc.) para mantener coherencia eventual entre motores.
* Documentar **runbooks** de incidente (quién, escalado, comunicación).

> *“La arquitectura avanzada es simple si cada pieza tiene dueño y SLO.”*
