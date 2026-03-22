---
layout: default
---

# Fundamento: errores, recuperación y arquitecturas híbridas
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-14](../../../sesiones/sesion-14.md)

*(Alineado con la teoría de la Semana 14: [teoria14-bbdd.md](../teoria/teoria14-bbdd.md).)*

---

## 1. Operaciones de alto riesgo

`DROP`, `TRUNCATE` sin backup; políticas de **dry-run**, revisión en cuatro ojos, permisos limitados en producción.

---

## 2. Recuperación

Backups completos/incrementales, **snapshots**, point-in-time recovery; pruebas periódicas de restauración.

---

## 3. Arquitecturas híbridas

Combinar SQL transaccional, NoSQL operacional y vector DB para IA; **CDC** y triggers para sincronizar sistemas.

---

## 4. Nube

Costes de egress, multi-región, **IaC** (Terraform) para reproducibilidad.

---

## 5. Enlaces directos

* **Teoría completa:** [teoria14-bbdd.md](../teoria/teoria14-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)

---

## 6. Cierre de curso

Integrar modelado, SQL, pipelines, big data, NoSQL y vectores con prácticas de **resiliencia** y gobierno.
