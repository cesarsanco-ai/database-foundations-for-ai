---
layout: default
---

# Fundamento: despliegue, contenedores, replicación y alta disponibilidad
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-13](../../../sesiones/sesion-13.md)

*(Alineado con la teoría de la Semana 13: [teoria13-bbdd.md](../teoria/teoria13-bbdd.md).)*

---

## 1. Contenedores

Imagen Docker con motor + versión fija; **volúmenes** para persistencia; variables de entorno para secretos vía orquestador.

---

## 2. Orquestación (Kubernetes)

StatefulSets, servicios, **health checks**, límites CPU/RAM; operadores de bases gestionadas en cloud.

---

## 3. Alta disponibilidad

Réplicas de lectura, **failover**, quorum; RPO/RTO objetivos de negocio.

---

## 4. Replicación

Streaming lógico/físico según motor; **lag** de réplica y conmutación automática.

---

## 5. Migraciones

Dump/restore, replicación continua, dual-write temporal — minimizar downtime.

---

## 6. Enlaces directos

* **Teoría completa:** [teoria13-bbdd.md](../teoria/teoria13-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)
