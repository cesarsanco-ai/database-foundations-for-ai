---
layout: default
---

# Fundamento: estructuras de datos en motores SQL y NoSQL
#### Autor: Carlos César Sánchez Coronel

[⬅️ Volver a la Sesión-03](../../../sesiones/sesion-03.md)

*(Alineado con la teoría de esta semana en el repositorio: [teoria2-bbdd.md](../../Sesion04/teoria/teoria2-bbdd.md) — *Estructuras de datos en SQL y NoSQL*; la carpeta `Sesion03` no tiene subcarpeta `teoria` duplicada.)*

---

## 1. Índices

Estructura auxiliar para acelerar búsquedas a costa de espacio y coste en escritura. Sin índice: búsqueda **O(n)**; con B-Tree típico: **O(log n)** en condiciones ideales.

---

## 2. B-Tree / B+Tree

Estructuras estándar en motores relacionales; nodos balanceados, optimizadas para **disco** (páginas).

---

## 3. Motores SQL vs NoSQL

| Familia | Estructura típica | Consultas |
|---------|-------------------|-----------|
| Relacional | B+Tree, heap | SQL, joins |
| Documento | B-Tree + documentos | Por clave / índice secundario |
| Columna ancha | Partición + columnas | Agregaciones wide |
| Clave-valor | Tablas hash | Get/Put por clave |

---

## 4. Implicaciones

Elección de **clave primaria**, índices compuestos y tipos de datos afecta latencia de lecturas y mantenimiento de índices en **INSERT/UPDATE/DELETE**.

---

## 5. Enlaces directos

* **Teoría (documento principal):** [teoria2-bbdd.md](../../Sesion04/teoria/teoria2-bbdd.md)
* **CheatSheet:** [cheatsheet.md](../cheatsheet/cheatsheet.md)

---

## 6. Preguntas de diseño

¿Acceso por rango o por igualdad? ¿Patrón de escritura intenso? → deciden tipo de índice y particionamiento.
