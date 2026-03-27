---
layout: default
---

# Cheatsheet: Modelamiento
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-02](../../../sesiones/sesion-02.md)

---

## 🎯 ¿Qué es?

Diseñar cómo se almacenan los datos **sin redundancia**, **consistentes** y alineados al negocio.

---

## 🔄 Ciclo de Modelamiento

```
Conceptual → Lógico → Físico
(E-R)        (Tablas)   (SQL real)
```

* **Conceptual:** idea del negocio
* **Lógico:** estructura (tablas, PK, FK)
* **Físico:** implementación (tipos, índices)

---

## 🧩 Modelo Conceptual (E-R)

### Entidades

* Objeto del mundo real (`Cliente`)
* **Fuerte:** independiente
* **Débil:** depende de otra

### Atributos

* **Simples:** indivisibles (`edad`)
* **Compuestos:** se dividen (`dirección`)
* **Multivalor:** varios valores (`teléfonos`)
* **Derivados:** calculados (`edad`)
* **Clave:** identificador único (PK)

### Relaciones

* **1:1** → uno a uno
* **1:N** → uno a muchos
* **N:M** → muchos a muchos (requiere tabla intermedia)

### Participación

* **Total:** obligatoria
* **Parcial:** opcional

---

## 🧱 Modelo Lógico

### Reglas clave

* Entidad → **Tabla**
* Atributo → **Columna**
* Clave → **PK**
* 1:N → FK en lado N
* N:M → tabla intermedia
* Multivalor → tabla aparte

---

## 🧼 Normalización

| Forma    | Regla                       | Objetivo                      |
| -------- | --------------------------- | ----------------------------- |
| **1FN**  | Datos atómicos              | Sin listas/repeticiones       |
| **2FN**  | Dependen de toda la PK      | Evitar dependencias parciales |
| **3FN**  | Sin dependencias indirectas | Evitar redundancia            |
| **BCNF** | Más estricta que 3FN        | Consistencia total            |
| **4FN**  | Multivalores separados      | Evitar duplicidad             |
| **5FN**  | Dependencias complejas      | Casos avanzados               |

👉 En la práctica: **hasta 3FN/BCNF es suficiente**

---

## ⚙️ Modelo Físico

### Tipos de datos comunes

* `INT` → enteros
* `VARCHAR(n)` → texto variable
* `TEXT` → texto largo
* `DATE / TIMESTAMP` → fechas
* `DECIMAL` → dinero (preciso)
* `FLOAT` → aproximado
* `BOOLEAN` → true/false
* `JSON` → datos semi-estructurados

---

## 🔗 Relaciones en práctica

* **1:N:** FK en tabla hija
* **N:M:** tabla intermedia
* **Débil:** PK incluye FK

---

## 🧠 Ejemplos típicos

* Estudiante ↔ Curso → N:M → `Matricula`
* Pedido ↔ Producto → N:M → `DetallePedido`
* Usuario ↔ Usuario → N:M → `Sigue`

---

## 🔐 ACID (Transacciones)

* **A**tomicidad → todo o nada
* **C**onsistencia → datos válidos
* **I**solation → no interferencia
* **D**urabilidad → no se pierde

---

## 🔄 Niveles de Aislamiento

1. **Read Uncommitted** → sucio 😬
2. **Read Committed** → seguro básico
3. **Repeatable Read** → consistente
4. **Serializable** → máximo (más lento)

---

## 🔒 Locks

* **Shared (S):** lectura
* **Exclusive (X):** escritura
* ⚠️ Puede haber **deadlocks**

---

## ⚡ Reglas de oro

* Evita redundancia
* Diseña antes de programar
* Usa N:M correctamente
* Normaliza (mínimo 3FN)
* Piensa en consultas reales

