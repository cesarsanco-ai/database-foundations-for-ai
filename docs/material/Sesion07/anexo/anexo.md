---
layout: default
---

# Fundamento: vistas, rutinas, triggers y lógica en el servidor
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-07](../../../sesiones/sesion-07.md)

*(Alineado con la teoría de la Semana 7: [teoria7-bbdd.md](../teoria/teoria7-bbdd.md).)*

---

## 1. Vistas

Tablas virtuales sobre `SELECT`; **vistas materializadas** para precomputar (refresco periódico). Uso: seguridad, simplificación, desacoplar aplicaciones.

---

## 2. Funciones y procedimientos

Funciones retornan valores escalares o tablas; procedimientos encapsulan secuencias transaccionales. Sintaxis varía (PostgreSQL, SQL Server, Oracle).

---

## 3. Triggers

Reacción a `INSERT/UPDATE/DELETE`; útiles para auditoría y reglas de negocio, pero riesgo de **complejidad oculta** y rendimiento.

---

## 4. Permisos

`GRANT` sobre vistas y rutinas para exponer solo lo necesario (**principio de mínimo privilegio**).

---

## 5. Enlaces directos

* **Teoría completa:** [teoria7-bbdd.md](../teoria/teoria7-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
