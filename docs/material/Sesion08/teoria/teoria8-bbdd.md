---
layout: default
---
# Sesion 8: Programacion en el servidor con PostgreSQL

### 1. Logro de la sesion

Disenar logica de negocio dentro del motor PostgreSQL usando vistas, funciones, procedimientos, triggers y politicas basicas de seguridad para automatizar procesos y mejorar la consistencia operacional.

---

### 2. Conexion con el syllabus

Semana 8 corresponde a:

- Views: abstraccion y seguridad.
- Stored procedures y functions.
- Triggers: automatizacion y auditoria.

Se alinea con `8-programacion.sql`.

---

### 3. Que es programar en el servidor

Es mover parte de la logica al motor de base de datos para:

1. centralizar reglas,
2. reducir duplicidad en aplicaciones,
3. asegurar integridad cercana al dato.

No reemplaza la aplicacion, la complementa.

---

### 4. Vistas (`VIEW`)

Una vista encapsula una consulta reusable.

```sql
CREATE VIEW vista_usuarios_activos AS
SELECT id, username, email, pais, ciudad, creado_en
FROM usuarios;
```

Beneficios:

- simplifica consultas,
- estandariza metricas,
- facilita control de acceso por capas.

---

### 5. Vistas con joins

```sql
CREATE VIEW vista_resumen_actividad AS
SELECT
    u.id,
    u.username,
    COUNT(DISTINCT s.id) AS total_sesiones,
    COUNT(DISTINCT m.id) AS total_mensajes
FROM usuarios u
LEFT JOIN sesiones s ON u.id = s.usuario_id
LEFT JOIN mensajes m ON u.id = m.remitente_id
GROUP BY u.id, u.username;
```

Uso tipico:

- dashboards,
- consultas de negocio consistentes.

---

### 6. Vistas materializadas

Guardan resultado fisico para acelerar lectura.

```sql
CREATE MATERIALIZED VIEW mv_resumen_diario AS
SELECT DATE(enviado_en) AS fecha, modalidad, COUNT(*) AS total
FROM mensajes
GROUP BY DATE(enviado_en), modalidad;
```

Refresco:

```sql
REFRESH MATERIALIZED VIEW mv_resumen_diario;
```

---

### 7. Funciones escalares

Devuelven un valor.

```sql
CREATE OR REPLACE FUNCTION tasa_exito_modulo(p_modulo TEXT)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
DECLARE
    v_total INTEGER;
    v_ok INTEGER;
BEGIN
    SELECT COUNT(*), COUNT(*) FILTER (WHERE exito = TRUE)
    INTO v_total, v_ok
    FROM logs_ejecucion
    WHERE modulo = p_modulo;

    IF v_total = 0 THEN
        RETURN NULL;
    END IF;

    RETURN ROUND((v_ok::NUMERIC / v_total) * 100, 2);
END;
$$;
```

---

### 8. Funciones tipo tabla

Permiten devolver conjuntos estructurados.

```sql
CREATE OR REPLACE FUNCTION mensajes_usuario(p_usuario_id INTEGER)
RETURNS TABLE(mensaje_id INTEGER, contenido TEXT, modalidad TEXT, enviado_en TIMESTAMPTZ)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT id, contenido, modalidad, enviado_en
    FROM mensajes
    WHERE remitente_id = p_usuario_id
    ORDER BY enviado_en DESC;
END;
$$;
```

---

### 9. Procedimientos almacenados

A diferencia de funciones, se invocan con `CALL` y orientan procesos.

```sql
CREATE OR REPLACE PROCEDURE limpiar_sesiones_expiradas()
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE sesiones
    SET cerrada = TRUE
    WHERE expira_en < NOW() AND cerrada = FALSE;
END;
$$;
```

```sql
CALL limpiar_sesiones_expiradas();
```

---

### 10. Manejo de excepciones en PL/pgSQL

```sql
EXCEPTION
    WHEN OTHERS THEN
        RAISE NOTICE 'Error: %', SQLERRM;
```

Usar para:

- registrar errores,
- proteger flujos criticos,
- devolver estados controlados.

---

### 11. Triggers: automatizacion por evento

Un trigger ejecuta logica ante `INSERT`, `UPDATE` o `DELETE`.

Componentes:

1. funcion trigger,
2. objeto trigger,
3. evento y momento (`BEFORE/AFTER`).

---

### 12. Trigger de actualizacion automatica

```sql
CREATE OR REPLACE FUNCTION actualizar_ultima_actividad()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE sesiones
    SET ultima_actividad = NEW.enviado_en
    WHERE id = NEW.sesion_id;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trig_actualizar_actividad
AFTER INSERT ON mensajes
FOR EACH ROW
EXECUTE FUNCTION actualizar_ultima_actividad();
```

---

### 13. Trigger para auditoria

Patron comun:

- tabla de auditoria,
- trigger `AFTER UPDATE`,
- registro de valores antes/despues.

Beneficios:

- trazabilidad,
- cumplimiento,
- analisis forense.

---

### 14. Trigger de validacion (`BEFORE`)

Ideal para bloquear datos invalidos antes de persistir.

Ejemplo:

- token duplicado,
- formato invalido,
- reglas minimas de longitud.

---

### 15. Seguridad basica en servidor

Elementos clave:

- roles,
- grants,
- vistas seguras,
- `SECURITY DEFINER` con criterio.

Principio:

aplicar minimo privilegio.

---

### 16. Row Level Security (RLS)

Permite filtrar filas por usuario/politica.

Uso recomendado para entornos multi-tenant.

Advertencia:

RLS mal configurado puede bloquear o exponer datos.

---

### 17. Cuándo llevar logica al servidor

Si la regla es:

- transversal a varias apps,
- critica para integridad,
- sensible a concurrencia,

entonces conviene server-side.

Si es presentacion de UI, conviene app-side.

---

### 18. Rendimiento y mantenibilidad

Buenas practicas:

1. evitar triggers innecesarios en rutas calientes,
2. instrumentar tiempos de ejecucion,
3. versionar funciones y procedimientos,
4. cubrir con pruebas SQL.

---

### 19. Errores frecuentes

- mezclar logica de negocio compleja sin documentacion,
- crear triggers que se disparan en cascada sin control,
- abuso de `SECURITY DEFINER`,
- no auditar cambios en objetos de base,
- desplegar funciones sin rollback.

---

### 20. Mini laboratorio

1. Crear una vista de resumen de actividad.
2. Crear funcion de tasa de exito por modulo.
3. Crear procedimiento de cierre de sesiones.
4. Implementar trigger de auditoria de usuarios.
5. Probar permisos con dos roles.

---

### 21. Checklist de salida

- Creo vistas y materialized views con criterio.
- Escribo funciones y procedimientos en PL/pgSQL.
- Implemento triggers para automatizacion y auditoria.
- Entiendo diferencia entre seguridad de app y de BD.
- Relaciono programacion servidor con calidad de datos.

---

### 22. Preguntas de autoevaluacion

1. Que diferencia practica hay entre funcion y procedimiento?
2. Cuando usar `AFTER INSERT` vs `BEFORE INSERT`?
3. Que riesgo tiene `SECURITY DEFINER`?
4. Cuando conviene materialized view?
5. Que regla deberia vivir siempre en base de datos?

---

### 23. Referencias recomendadas

1. PostgreSQL docs: PL/pgSQL.
2. PostgreSQL docs: CREATE VIEW.
3. PostgreSQL docs: CREATE TRIGGER.
4. PostgreSQL docs: RLS.
5. Scripts practicos de programacion del curso.
