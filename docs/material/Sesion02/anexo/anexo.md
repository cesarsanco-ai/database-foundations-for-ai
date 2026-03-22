---
layout: default
---

# Fundamento: modelamiento entidad–relación y niveles de diseño
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-02](../../../sesiones/sesion-02.md)

*(Alineado con la teoría de la Semana 2: [teoria2-bbdd.md](../teoria/teoria2-bbdd.md).)*

---

## 1. Ciclo de modelamiento

**Conceptual** (E-R) → **Lógico** (tablas normalizadas) → **Físico** (tipos, índices, particiones).

---

## 2. Modelo E-R

Entidades, atributos, cardinalidades (1:1, 1:N, N:M). El diagrama comunica negocio y equipo técnico.

---

## 3. Normalización (introducción)

Formas normales para reducir redundancia y anomalías de actualización; profundización en la Semana 4.

---

## 4. Modelo físico

Tipos de datos, claves, índices, **ACID**, aislamiento, locks — puente hacia el motor concreto (PostgreSQL, etc.).

---

## 5. NoSQL y CAP (visión)

Cuándo documento vs columna vs grafo; **teorema CAP** como marco de compromisos en sistemas distribuidos.

---

## 6. Enlaces directos

* **Teoría completa:** [teoria2-bbdd.md](../teoria/teoria2-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)

---

## 7. Buenas prácticas

Nombres consistentes, documentar supuestos, validar cardinalidades con stakeholders antes de crear tablas.
