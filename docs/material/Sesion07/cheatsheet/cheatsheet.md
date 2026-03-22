---
layout: default
---

# Cheatsheet: Programación en el servidor
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-07](../../../sesiones/sesion-07.md)

---

## Objetos

| Objeto | Rol típico |
| :--- | :--- |
| **VIEW** | Consulta reutilizable / seguridad |
| **FUNCTION** | Cálculo puro o tabla |
| **PROCEDURE** | Flujo con transacciones |
| **TRIGGER** | Evento DML → acción |

---

## Vista (PostgreSQL-style)

```sql
CREATE VIEW activos AS
SELECT id, nombre FROM cliente WHERE activo;

GRANT SELECT ON activos TO app_reader;
```

---

## Trigger (idea)

`BEFORE INSERT` para validar; `AFTER` para auditoría — evitar lógica pesada por fila.

---

## Teoría

* [teoria7-bbdd.md](../teoria/teoria7-bbdd.md)

---

## Puntos críticos

* Triggers en cascada complican **depuración**.
* Versionar rutinas como **código** (migraciones), no solo en consola.

> *“Lógica en el servidor reduce round-trips; también acopla al motor.”*
