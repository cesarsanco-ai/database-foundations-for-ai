---
layout: default
---

# Fundamento: normalización e integridad técnica
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-04](../../../sesiones/sesion-04.md)

*(Alineado con las secciones de normalización, integridad y transacciones en [teoria2-bbdd.md](../../Sesion02/teoria/teoria2-bbdd.md).)*

---

## 1. Formas normales (idea)

- **1FN:** atributos atómicos (sin listas repetidas en una celda).
- **2FN:** 1FN + sin dependencias parciales de claves compuestas.
- **3FN:** 2FN + sin dependencias transitivas de la clave.

Objetivo: reducir **redundancia** y **anomalías** de inserción/actualización/borrado.

---

## 2. Claves y restricciones

**PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL** — el motor garantiza reglas declarativamente.

---

## 3. Integridad referencial

Borrados/actualizaciones en cascada o restricciones según política de negocio (`ON DELETE CASCADE`, etc.).

---

## 4. Transacciones y ACID

**Atomicidad, Consistencia, Aislamiento, Durabilidad.** Niveles de **aislamiento** (READ COMMITTED, SERIALIZABLE, …) equilibran consistencia vs rendimiento.

---

## 5. Enlaces directos

* **Teoría (mismo documento que modelamiento; secciones FN y ejercicios):** [teoria2-bbdd.md](../../Sesion02/teoria/teoria2-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)

---

## 6. Desnormalización controlada

A veces se acepta redundancia por rendimiento (reporting, NoSQL); debe documentarse y vigilarse consistencia.
