---
layout: default
---

# Cheatsheet: Programación en el servidor
**Autor:** Carlos César Sánchez Coronel  

[⬅️ Volver a la Sesión-07](../../../sesiones/sesion-07.md)

---

## 📌 1. VISTAS (VIEWS)

### ✔️ Concepto

* Tabla virtual basada en `SELECT`
* No almacena datos (excepto materializadas)

### 🔹 Crear vista

```sql
CREATE VIEW clientes_activos AS
SELECT id, nombre FROM clientes WHERE activo = true;
```

### 🔹 Usar

```sql
SELECT * FROM clientes_activos;
```

### 🔹 Casos de uso

* Seguridad (ocultar columnas)
* Simplificar queries complejas
* Reutilización lógica

---

## ⚡ Vistas Materializadas

### ✔️ Guardan datos físicamente

```sql
CREATE MATERIALIZED VIEW resumen AS
SELECT producto_id, SUM(cantidad) FROM ventas GROUP BY producto_id;
```

### 🔄 Refrescar

```sql
REFRESH MATERIALIZED VIEW resumen;
```

### 🧠 Cuándo usar

* Reportes pesados
* Datos no en tiempo real

---

## ⚙️ 2. FUNCIONES

### ✔️ Concepto

* Retornan valor (escalar o tabla)
* Usables dentro de queries

---

### 🔹 Función escalar

```sql
CREATE FUNCTION calcular_iva(monto NUMERIC)
RETURNS NUMERIC AS $$
BEGIN
  RETURN monto * 0.21;
END;
$$ LANGUAGE plpgsql;
```

```sql
SELECT calcular_iva(100);
```

---

### 🔹 Función tabla

```sql
CREATE FUNCTION empleados_por_depto(id INT)
RETURNS TABLE(nombre TEXT, salario NUMERIC) AS $$
BEGIN
  RETURN QUERY SELECT nombre, salario FROM empleados WHERE departamento_id = id;
END;
$$ LANGUAGE plpgsql;
```

---

### 🧠 Claves

* No manejan `COMMIT/ROLLBACK`
* Ideales para lógica reutilizable

---

## 🏗️ 3. PROCEDIMIENTOS

### ✔️ Concepto

* Ejecutan acciones (INSERT, UPDATE…)
* Soportan transacciones

---

### 🔹 Crear procedimiento

```sql
CREATE PROCEDURE transferir(origen INT, destino INT, monto NUMERIC)
LANGUAGE plpgsql AS $$
BEGIN
  UPDATE cuentas SET saldo = saldo - monto WHERE id = origen;
  UPDATE cuentas SET saldo = saldo + monto WHERE id = destino;
  COMMIT;
END;
$$;
```

### 🔹 Ejecutar

```sql
CALL transferir(1,2,100);
```

---

### 🧠 Diferencia clave

| Función           | Procedimiento     |
| ----------------- | ----------------- |
| Retorna valor     | No necesariamente |
| Usable en SELECT  | Se llama con CALL |
| Sin transacciones | Con transacciones |

---

## 🔥 4. TRIGGERS

### ✔️ Concepto

Código automático ante eventos

---

### 🔹 Tipos

* `BEFORE` → validar/modificar
* `AFTER` → auditoría
* `INSTEAD OF` → vistas

---

### 🔹 Ejemplo auditoría

```sql
CREATE FUNCTION audit_func()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit(tabla, fecha) VALUES ('empleados', NOW());
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_trigger
AFTER UPDATE ON empleados
FOR EACH ROW EXECUTE FUNCTION audit_func();
```

---

### 🔹 Validación

```sql
IF NEW.salario < 0 THEN
  RAISE EXCEPTION 'Salario inválido';
END IF;
```

---

### ⚠️ Riesgos

* Impacto en rendimiento
* Difíciles de depurar
* Posible recursividad

---

## 🔁 5. TRANSACCIONES

```sql
BEGIN;
UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;
```

### ❌ Rollback

```sql
ROLLBACK;
```

---

## ⚠️ 6. EXCEPCIONES

```sql
BEGIN
  RETURN a / b;
EXCEPTION
  WHEN division_by_zero THEN
    RETURN NULL;
END;
```

---

## 🔄 7. CURSORES (EVITAR SI ES POSIBLE)

```sql
FOR r IN SELECT * FROM empleados LOOP
  -- lógica fila por fila
END LOOP;
```

### ⚠️ Mejor usar:

```sql
UPDATE empleados SET salario = salario * 1.05;
```

---

## 🔐 8. SEGURIDAD

### 🔹 Hash (contraseñas)

```sql
SELECT md5('password');
```

```sql
SELECT encode(digest('password','sha256'),'hex');
```

---

### 🔹 Buenas prácticas

* Usar **hash + salt**
* NO guardar contraseñas en texto plano
* Preferir bcrypt (en backend)

---

### 🔹 Permisos

```sql
GRANT EXECUTE ON FUNCTION mi_funcion TO app_user;
```

---

### 🔹 SECURITY DEFINER

```sql
CREATE FUNCTION eliminar_usuario()
SECURITY DEFINER
```

👉 Ejecuta con permisos del creador

---

## 🔗 9. INTEGRACIÓN CON APLICACIONES

### ✔️ Patrón típico

* App → función/procedimiento → BD

---

### 🔹 Ejemplo (Node.js)

```javascript
await client.query(
  'SELECT generar_ticket($1, $2)',
  [5, 45.5]
);
```

---

### 🔹 Ejemplo (Python)

```python
cur.callproc('generar_ticket', [5,45.5])
```

---

### 🧠 Recomendación

* NO conectar apps directamente a BD (usar API)

---

## 🤖 10. INTEGRACIÓN CON IA / ML

### 🔹 Dataset desde BD

```sql
SELECT * FROM datos_entrenamiento();
```

---

### 🔹 Beneficios

* Menos transferencia de datos
* Features consistentes

---

## ⚙️ 11. ETL / AUTOMATIZACIÓN

### 🔹 Procedimiento ETL

```sql
CALL actualizar_resumen();
```

---

### 🔹 Airflow

```python
PostgresOperator(sql="CALL actualizar_resumen();")
```

---

## ⚡ 12. OPTIMIZACIÓN

### ✔️ Buenas prácticas

* Evitar cursores
* Usar operaciones en conjunto
* Indexar columnas usadas
* Analizar con `EXPLAIN`

---

### ❌ Mal

```sql
FOR r IN SELECT salario LOOP ...
```

### ✅ Bien

```sql
SELECT SUM(salario) FROM empleados;
```

---

## 🧩 13. CUÁNDO USAR CADA UNO

| Caso                  | Usar                |
| --------------------- | ------------------- |
| Simplificar consultas | Vista               |
| Reportes pesados      | Vista materializada |
| Lógica reutilizable   | Función             |
| Procesos complejos    | Procedimiento       |
| Automatización        | Trigger             |
| Seguridad             | Función + roles     |

---

## 🚀 REGLAS DE ORO

* 🧠 “Lleva la lógica cerca de los datos”
* ⚡ Usa SQL set-based (no loops)
* 🔐 Aplica mínimo privilegio
* 🔍 Mide rendimiento siempre
* ⚖️ Balancea BD vs aplicación

