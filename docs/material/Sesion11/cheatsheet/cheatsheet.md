---
layout: default
---

# Cheatsheet: Ecosistema NoSQL
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-11](../../../sesiones/sesion-11.md)

---

## Familias

| Familia | Ejemplo | Bueno para |
| :--- | :--- | :--- |
| Documento | MongoDB | Catálogos, perfiles |
| Clave-valor | Redis | Caché, sesiones |
| Columna | Cassandra | Series temporales masivas |
| Grafo | Neo4j | Redes, recomendación relacional |

---

## CAP (recordatorio)

No se pueden maximizar **C+A+P** a la vez en presencia de particiones — diseñar compromiso explícito.

---

## Teoría

* [teoria11-bbdd.md](../teoria/teoria11-bbdd.md)

---

## Mongo (idea)

```javascript
db.pedidos.find({ clienteId: ObjectId("...") }).sort({ fecha: -1 })
```

---

## Puntos críticos

* **Índices** en NoSQL siguen siendo críticos; sin ellos, scans completos.
* Migrar de SQL a documento sin rediseño ⇒ rendimiento pobre.

> *“Elige el motor según el patrón de lectura/escritura, no según la moda.”*
