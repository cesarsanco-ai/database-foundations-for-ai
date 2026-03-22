---
layout: default
---

# Cheatsheet: Universo del dato
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-01](../../../sesiones/sesion-01.md)

---

## Tipos de datos (recordatorio)

| Escala | Ejemplos | Operaciones típicas |
| :--- | :--- | :--- |
| Nominal | ID, color | =, ≠ |
| Ordinal | NPS, nivel | orden |
| Continuo | precio, temp. | +, media, modelado |

---

## Pipeline mental

1. Origen → 2. Calidad → 3. Modelo lógico/físico → 4. Uso analítico / ML

---

## Infra (macro)

| Componente | Pregunta |
| :--- | :--- |
| Almacenamiento | ¿Dónde persisten los bytes? |
| Red | ¿Quién accede y con qué latencia? |
| Backup | ¿RPO / RTO aceptables? |

---

## Teoría

* [teoria1-bbdd.md](../teoria/teoria1-bbdd.md)

---

## Puntos críticos

* **Garbage in, garbage out:** reglas de negocio en el motor no arreglan datos mal capturados.
* Documentar **unidades** y **zonas horarias** en datos temporales.

> *“El dato es el activo; el resto es procesamiento.”*
