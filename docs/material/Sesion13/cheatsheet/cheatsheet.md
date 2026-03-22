---
layout: default
---

# Cheatsheet: Despliegue y alta disponibilidad
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-13](../../../sesiones/sesion-13.md)

---

## Conceptos

| Término | Significado |
| :--- | :--- |
| **RPO** | Pérdida máxima de datos aceptable |
| **RTO** | Tiempo máximo para volver a servir |
| **Lag** | Retraso réplica primaria → secundaria |

---

## Docker (esqueleto)

```dockerfile
FROM postgres:16
ENV POSTGRES_PASSWORD=changeme
VOLUME /var/lib/postgresql/data
EXPOSE 5432
```

---

## HA (ideas)

* Réplicas read-only + balanceador  
* Failover automático (Patroni, operadores cloud)  
* Backups + **PITR** probados  

---

## Teoría

* [teoria13-bbdd.md](../teoria/teoria13-bbdd.md)

---

## Puntos críticos

* Contenedor **sin volumen** = pérdida de datos al borrar.
* Probar restore **antes** del incidente.

> *“La alta disponibilidad es un proceso: runbooks + pruebas de caos controladas.”*
