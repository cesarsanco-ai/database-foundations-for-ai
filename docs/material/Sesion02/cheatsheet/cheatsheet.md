---
layout: default
---

# Cheatsheet: Modelamiento
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-02](../../../sesiones/sesion-02.md)

---

## Niveles

| Nivel | Entrega |
| :--- | :--- |
| Conceptual | Diagrama E-R |
| Lógico | Esquema relacional |
| Físico | DDL + índices |

---

## Relaciones

| Cardinalidad | Tabla puente |
| :--- | :--- |
| 1:N | FK en lado N |
| N:M | Tabla intermedia |

---

## Checklist E-R

* Entidades = sustantivos de negocio  
* Atributos = propiedades atómicas (1FN)  
* Identificadores únicos claros  

---

## Teoría

* [teoria2-bbdd.md](../teoria/teoria2-bbdd.md)

---

## SQL (boceto DDL)

```sql
CREATE TABLE pedido (
  id UUID PRIMARY KEY,
  cliente_id UUID NOT NULL REFERENCES cliente(id),
  fecha TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## Puntos críticos

* **N:M** mal modelado genera duplicados o tablas anámicas imposibles de consultar.
* Pensar en **consultas frecuentes** ya en diseño lógico (no solo en físico).

> *“El modelo conceptual es el contrato con el negocio.”*
