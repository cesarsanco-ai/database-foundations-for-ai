---
layout: default
---

# Cheatsheet: Estructura de datos en base de datos
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-03](../../../sesiones/sesion-03.md)

---

## 🎯 ¿Por qué normalizar?

Evitar problemas:

* ❌ Inserción → no puedes guardar datos incompletos
* ❌ Actualización → inconsistencias
* ❌ Eliminación → pérdida accidental

👉 Objetivo: **consistencia + sin redundancia**

---

## 🔗 Dependencias Funcionales (DF)

👉 Regla:

```
X → Y  (X determina Y)
```

* Si conoces **X**, sabes **Y**
* Base de la normalización

Ej:

```
id_departamento → nombre_departamento
```

---

## 🧼 Formas Normales

### 🥇 1NF (Primera Forma Normal)

* Datos **atómicos**
* ❌ listas / arrays en columnas

✔ Solución:

* Separar en otra tabla

---

### 🥈 2NF (Segunda Forma Normal)

* Cumple 1NF
* No hay dependencias parciales

👉 Problema:

* PK compuesta
* Un atributo depende solo de una parte

✔ Solución:

* Crear nueva tabla

---

### 🥉 3NF (Tercera Forma Normal)

* Cumple 2NF
* ❌ dependencias transitivas

👉 Problema:

```
A → B → C
```

✔ Solución:

* Separar en otra tabla

---

### 🧠 BCNF

* Más estricta que 3NF
* Todo determinante debe ser **superclave**

---

### 🚀 4NF y 5NF

* 4NF → multivalores
* 5NF → joins complejos

👉 En la práctica: **hasta 3NF / BCNF**

---

## ⚠️ Tipos de anomalías

* **Inserción** → falta info
* **Actualización** → duplicación
* **Eliminación** → pérdida de datos

---

## 🔐 Integridad en SQL

### 1. Dominio (valores válidos)

* `NOT NULL` → obligatorio
* `CHECK` → condición
* Tipos (`INT`, `DATE`, etc.)

---

### 2. Entidad (unicidad)

* `PRIMARY KEY` → único + no null
* `UNIQUE` → único

---

### 3. Referencial (relaciones)

* `FOREIGN KEY`

Opciones:

* `CASCADE` → propaga
* `SET NULL` → pone null
* `RESTRICT` → bloquea

---

### 4. Reglas extra

* `CHECK`, `UNIQUE`, combinaciones

---

### 5. Triggers

* Código automático (INSERT, UPDATE, DELETE)
* Para lógica compleja

---

## ⚖️ Normalización vs Desnormalización

| Normalización       | Desnormalización  |
| ------------------- | ----------------- |
| ✔ Consistencia      | ✔ Rendimiento     |
| ✔ Menos redundancia | ❌ Más duplicación |
| ❌ Más JOINs         | ✔ Menos JOINs     |

👉 Uso típico:

* OLTP → normalizado (3NF)
* Data warehouse → desnormalizado

---

## 🧩 Patrón clásico (ejemplo mental)

❌ Mal diseño:

```
Pedido(cliente, telefono, producto, precio)
```

✔ Bien:

```
Cliente(id, nombre, telefono)
Producto(id, nombre, precio)
Pedido(id, id_cliente)
DetallePedido(id_pedido, id_producto)
```

---

## ⚡ Reglas de oro

* Siempre 1NF → 2NF → 3NF
* Detecta dependencias funcionales
* Usa PK y FK correctamente
* Prefiere restricciones SQL > triggers
* Desnormaliza solo si hay problema real


