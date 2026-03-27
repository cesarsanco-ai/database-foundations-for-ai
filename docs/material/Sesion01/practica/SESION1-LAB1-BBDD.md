# 📘 Laboratorio 1: Analizar Caso de Estudio de un Asistente Multimodal

Este documento describe el **caso de negocio** que utilizaremos como hilo conductor a lo largo de todo el curso. Cada sesión práctica estará vinculada a este escenario ya que relaciona el concepto multimodal de la carrera en IA, permitiéndote aplicar los conceptos de base de datos sobre un problema realista y coherente.

---

## 🧠 Lógica del Negocio

**Aletza** es un asistente virtual inteligente al que los usuarios acceden a través de diferentes canales (Telegram, web, app). Su característica principal es la **autenticación por voz**: el usuario dice la palabra mágica **"aletza"** y, si el sistema reconoce su voz, se le da acceso a todas las funcionalidades del agente.

El sistema está pensado para **solo autenticación por voz**, sin alternativas de contraseña. El registro de voz se realiza una sola vez durante el proceso de alta.

---

## 👥 Actores y Roles

| Actor | Descripción |
|-------|-------------|
| **Usuario** | Persona que se registra en la plataforma, realiza el enrollment de voz (repite "aletza" 10 veces) y luego se registra por unica vez mediante su voz para usar el asistente, y ya luego cuando quiera usar solo dira una vez "aletza" para iniciar sesion. |
| **Agente / Bot** | El asistente IA multimodal que procesa los mensajes del usuario (texto, voz, imagen, video) utilizando módulos especializados: transcripción, análisis de imagen, clasificación de intención, generación de respuestas, etc. |

---

## 🔄 Flujo Completo

### 1. Registro y Enrollment de Voz
1. El ingresa a telegram desde el bot
2. Desde el bot de telegram, se le dare ciertas instrucciones de registro si es la primera vez que ingresa, el usuario envía **10 mensajes de voz** diciendo la palabra mágica **"aletza"**.
3. El sistema almacena cada muestra y entrena un perfil biométrico asociado al usuario (vector de voz).
4. Una vez completadas las 10 muestras, el perfil queda activo.

### 2. Autenticación por Voz (Uso Diario)
1. El usuario envía un mensaje de voz con la palabra "aletza" inicialmente para iniciar sesion.
2. El sistema extrae el vector de voz y lo compara con los perfiles almacenados.
3. Si hay coincidencia, se **inicia una sesión** y el agente responde:  
   *"Bienvenido [nombre], ¿en qué puedo ayudarte?"*
4. A partir de ese momento, el usuario puede enviar cualquier tipo de mensaje (texto, audio, imagen, video) y el agente responde usando los módulos de IA correspondientes.

### 3. Gestión de Sesiones (Timeout)
- Cada sesión tiene una duración máxima de **1 hora de inactividad**.
- Si el usuario no envía ningún mensaje durante 1 hora, la sesión se cierra automáticamente.
- Para volver a usar el asistente, el usuario debe decir nuevamente "aletza" y autenticarse de nuevo.

### 4. Uso del Agente
Una vez autenticado, el agente actúa como un **orquestador modular**. Cada mensaje del usuario pasa por varios módulos:

- **Multimodalidad:** Si el mensaje es audio, se transcribe; si es imagen, se describe; si es video, se analiza.
- **Clasificación NLP:** Se determina la intención (ej. "soporte técnico", "pedir información", "charla informal") y el sentimiento.
- **Generación de respuesta:** Se produce una respuesta coherente, personalizada y contextual.
- **Traza:** Cada paso queda registrado en logs de ejecución (qué módulo se usó, cuánto tardó, éxito/fallo).

---

## 🗂️ Modelo de Datos (Evolutivo)

A lo largo del curso iremos construyendo incrementalmente el modelo de datos. Las tablas principales serán:

```sql
-- Usuarios
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    email TEXT UNIQUE NOT NULL,
    creado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Perfiles de voz (enrollment)
CREATE TABLE perfiles_voz (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES usuarios(id),
    palabra_magica TEXT NOT NULL,          -- 'aletza'
    vector_voz JSONB,                      -- embeddings o datos biométricos
    muestras_tomadas INT DEFAULT 0,
    estado TEXT DEFAULT 'incompleto' CHECK (estado IN ('incompleto', 'completo'))
);

-- Sesiones autenticadas
CREATE TABLE sesiones (
    id SERIAL PRIMARY KEY,
    usuario_id INT REFERENCES usuarios(id),
    token TEXT UNIQUE NOT NULL,
    canal TEXT,
    iniciada_en TIMESTAMPTZ DEFAULT NOW(),
    ultima_actividad TIMESTAMPTZ DEFAULT NOW(),
    expira_en TIMESTAMPTZ DEFAULT NOW() + INTERVAL '1 hour'
);

-- Mensajes (conversación)
CREATE TABLE mensajes (
    id SERIAL PRIMARY KEY,
    sesion_id INT REFERENCES sesiones(id),
    remitente_id INT REFERENCES usuarios(id),
    contenido TEXT,                         -- texto original o transcripción
    modalidad TEXT CHECK (modalidad IN ('texto', 'audio', 'imagen', 'video')),
    metadata_ia JSONB,                      -- resultados de los módulos
    enviado_en TIMESTAMPTZ DEFAULT NOW()
);

-- Logs de ejecución de módulos (trazabilidad)
CREATE TABLE logs_ejecucion (
    id SERIAL PRIMARY KEY,
    mensaje_id INT REFERENCES mensajes(id),
    modulo TEXT NOT NULL,
    tiempo_ejecucion_ms INT,
    exito BOOLEAN,
    error TEXT,
    ejecutado_en TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 📝 Nota para el Estudiante

Este documento será tu **guía de referencia** durante todo el curso. Cada sesión retomará el caso de Aletza, añadiendo nuevos componentes o resolviendo problemas concretos. Te recomendamos conservarlo y tomar notas sobre las evoluciones que vayamos realizando.

**Palabra mágica:** **aletza**  
**Sesión:** dura 1 hora de inactividad; si se duerme, basta con decir "aletza" para despertar al asistente.

## Estructura visual

- Capa de entrada: webhook de telegram bot
- Capa de seguridad: Funcion auth_biometrica(), compara el audio recibido con perfilez_voz
- Capa de inteligencia: Modulos que escriben en metadata_ia (jsonb)
- Capa de observabilidad: La tabla logs_ejecucion que alimenta el dashboard de big data
---

