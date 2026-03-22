---
layout: default
---

# Cheatsheet: Estructura de datos en base de datos
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-03](../../../sesiones/sesion-03.md)

---

## Complejidad (orden de magnitud)

| Operación | Sin índice | Con B-Tree (idea) |
| :--- | :--- | :--- |
| Búsqueda por igualdad | O(n) | O(log n) |
| Inserción | O(1) append* | O(log n) mantener orden |

\*según motor y tabla

---

## Objetos

| Objeto | Función |
| :--- | :--- |
| **Índice** | Acelerar lecturas filtradas / joins |
| **PK** | Unicidad + acceso principal |
| **Índice compuesto** | Consultas multi-columna (orden importa) |

---

## Teoría

* [teoria2-bbdd.md](../../Sesion04/teoria/teoria2-bbdd.md)

---

## SQL (ejemplo índice)

```sql
CREATE INDEX idx_cliente_ciudad ON cliente (ciudad);
-- consultas: WHERE ciudad = 'Málaga'
```

---

## Puntos críticos

* **Índice mal elegido** = espacio desperdiciado y escrituras lentas.
* En NoSQL, muchos sistemas **no** ofrecen joins eficientes: modelar para **consultas**, no solo para normalización.

> *“El motor es rápido si el plan de ejecución usa el índice correcto.”*
