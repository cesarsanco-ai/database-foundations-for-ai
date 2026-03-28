# 📘 Laboratorio 2: ETL


## 📈 Paso 5: Vistas Analíticas para Consumo

### 5.1 Vistas en Capa Gold

```sql
-- Vista: Dashboard de métricas diarias
CREATE VIEW gold.vista_dashboard_diario AS
SELECT 
    fecha,
    total_usuarios_activos,
    total_sesiones,
    total_mensajes,
    total_mensajes / NULLIF(total_sesiones, 0) AS mensajes_por_sesion,
    tiempo_promedio_respuesta,
    tasa_exito_ia,
    mensajes_por_modalidad->>'texto' AS mensajes_texto,
    mensajes_por_modalidad->>'audio' AS mensajes_audio
FROM gold.resumen_diario
ORDER BY fecha DESC;

-- Vista: Top usuarios por actividad
CREATE VIEW gold.vista_top_usuarios AS
SELECT 
    u.usuario_id,
    u.username,
    u.pais,
    u.nivel_actividad,
    u.total_mensajes,
    COUNT(DISTINCT h.fecha) AS dias_activos,
    AVG(h.tiempo_procesamiento_ms) AS tiempo_promedio_procesamiento
FROM gold.dim_usuarios u
LEFT JOIN gold.hechos_mensajes h ON u.usuario_id = h.usuario_id
GROUP BY u.usuario_id, u.username, u.pais, u.nivel_actividad, u.total_mensajes
ORDER BY u.total_mensajes DESC
LIMIT 20;

-- Vista: Análisis de sentimiento por intención
CREATE VIEW gold.vista_sentimiento_intencion AS
SELECT 
    intencion,
    sentimiento,
    COUNT(*) AS cantidad,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY intencion), 2) AS porcentaje
FROM gold.hechos_mensajes
WHERE intencion IS NOT NULL AND sentimiento IS NOT NULL
GROUP BY intencion, sentimiento
ORDER BY intencion, cantidad DESC;

-- Vista: Horas de mayor actividad
CREATE VIEW gold.vista_actividad_por_hora AS
SELECT 
    EXTRACT(HOUR FROM hora) AS hora,
    COUNT(*) AS mensajes,
    COUNT(DISTINCT usuario_id) AS usuarios_activos
FROM gold.hechos_mensajes
GROUP BY EXTRACT(HOUR FROM hora)
ORDER BY hora;
```

### 5.2 Verificar Vistas Analíticas

```sql
-- Consultar dashboards
SELECT * FROM gold.vista_dashboard_diario LIMIT 10;
SELECT * FROM gold.vista_top_usuarios;
SELECT * FROM gold.vista_sentimiento_intencion;
SELECT * FROM gold.vista_actividad_por_hora;
```

---



### 6.2 Monitoreo de Pipelines

```sql
-- Vista de monitoreo de ejecuciones
CREATE VIEW gold.vista_monitoreo_pipelines AS
SELECT 
    proceso,
    ultima_ejecucion,
    registros_procesados,
    estado,
    fecha_carga,
    EXTRACT(EPOCH FROM (NOW() - ultima_ejecucion)) / 3600 AS horas_desde_ultima
FROM control_cargas
ORDER BY ultima_ejecucion DESC;

-- Verificar estado de pipelines
SELECT * FROM gold.vista_monitoreo_pipelines LIMIT 10;
```


---

## ✅ Verificación Final

```sql
-- Resumen de arquitectura implementada
SELECT 
    'Bronze (Raw)' AS capa,
    COUNT(*) AS tablas,
    (SELECT SUM(pg_total_relation_size(schemaname||'.'||tablename)) FROM pg_tables WHERE schemaname = 'bronze') AS tamaño_bytes
FROM pg_tables WHERE schemaname = 'bronze'
UNION ALL
SELECT 'Silver (Cleansed)', COUNT(*),
    (SELECT SUM(pg_total_relation_size(schemaname||'.'||tablename)) FROM pg_tables WHERE schemaname = 'silver')
FROM pg_tables WHERE schemaname = 'silver'
UNION ALL
SELECT 'Gold (Aggregated)', COUNT(*),
    (SELECT SUM(pg_total_relation_size(schemaname||'.'||tablename)) FROM pg_tables WHERE schemaname = 'gold')
FROM pg_tables WHERE schemaname = 'gold';

-- Verificar ejecución del pipeline
SELECT * FROM control_cargas ORDER BY id DESC LIMIT 5;
```

---

## 📝 Resumen de la Arquitectura Implementada

| Capa | Esquema | Propósito | Ejemplos |
|------|---------|-----------|----------|
| **Bronze** | `bronze.*_raw` | Datos crudos, sin transformar | `bronze.usuarios_raw`, `bronze.mensajes_raw` |
| **Silver** | `silver.*_cleansed` | Datos limpiados, normalizados | `silver.usuarios_cleansed`, `silver.mensajes_cleansed` |
| **Gold** | `gold.*` | Datos agregados, listos para análisis | `gold.hechos_mensajes`, `gold.resumen_diario` |

**Pipeline ETL:** 
```
Source (public) → Bronze (raw) → Silver (cleansed) → Gold (aggregated) → BI/ML
```

---

## 🎯 Conclusión

1. **Escalabilidad**: Separación clara de responsabilidades
2. **Trazabilidad**: Datos crudos preservados en Bronze
3. **Calidad**: Limpieza y normalización en Silver
4. **Rendimiento**: Agregaciones precalculadas en Gold
5. **Mantenibilidad**: Pipelines modulares y orquestados

Esta arquitectura permite alimentar dashboards, reportes y modelos de IA con datos confiables y actualizados.