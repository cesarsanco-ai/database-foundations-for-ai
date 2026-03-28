# 📘 Laboratorio 1: MongoDB Atlas

## 🎯 Objetivo

Migrar y adaptar los datos del asistente Aletza a MongoDB Atlas, aprovechando la flexibilidad de los documentos JSON para almacenar datos semiestructurados y mejorar el rendimiento en consultas analíticas.

Al finalizar, podrás:
- Crear un cluster gratuito en MongoDB Atlas
- Diseñar esquemas documentales para datos de usuarios, sesiones y mensajes
- Insertar y consultar datos usando la interfaz web de Atlas
- Realizar agregaciones complejas (equivalente a GROUP BY y JOINs en SQL)
- Optimizar consultas con índices en MongoDB

---

## 📋 Requisitos Previos

- Cuenta en MongoDB Atlas (gratuita)
- Navegador web moderno (Chrome, Firefox, Edge)
- Datos del caso Aletza (los mismos que usamos en PostgreSQL)

---

## 🌐 Paso 1: Creación del Ambiente en MongoDB Atlas

### 1.1 Crear una Cuenta Gratuita

1. Ir a [https://www.mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Hacer clic en **"Try Free"** o **"Start Free"**
3. Registrarse con Google, GitHub o email
4. Verificar el correo electrónico

### 1.2 Crear un Cluster Gratuito (M0)

1. Una vez dentro del panel, hacer clic en **"Build a Database"**

2. Seleccionar **"Shared Clusters"** (gratuito)

3. Seleccionar **"M0 Sandbox"** (opción gratuita)

4. Elegir un proveedor de nube:
   - **AWS** (recomendado para latencia en América Latina)

5. Seleccionar la región más cercana:
   - Para Perú: `sa-east-1` (São Paulo) o `us-east-1` (Virginia)

6. Dar un nombre al cluster:
   - Recomendado: `AletzaCluster`

7. Hacer clic en **"Create Cluster"**
   - Esperar 1-3 minutos mientras se crea el cluster

### 1.3 Configurar Usuario y Acceso

1. En la sección **"Security Quickstart"**:
   - **Username**: `aletza_user`
   - **Password**: Anotar la contraseña generada
   - **Authentication Method**: `Password`

2. En **"Where would you like to connect from?"**:
   - Seleccionar **"My Local Environment"**
   - Añadir tu IP actual (hacer clic en **"Add My Current IP Address"**)

3. Hacer clic en **"Finish and Close"**

### 1.4 Acceder a la Interfaz de Base de Datos

1. En el panel principal, hacer clic en **"Browse Collections"**

2. Verás la estructura:
   - **Organization**: tu organización
   - **Project**: proyecto actual
   - **Cluster**: `AletzaCluster`

---

## 🗄️ Paso 2: Creación de Base de Datos y Colecciones

### 2.1 Crear Base de Datos

1. En **"Browse Collections"**, hacer clic en **"Create Database"**

2. Configurar:
   - **Database Name**: `aletza_db`
   - **Collection Name**: `usuarios`
   - Hacer clic en **"Create"**

3. Repetir para crear las siguientes colecciones:
   - `perfiles_voz`
   - `sesiones`
   - `mensajes`
   - `logs_ejecucion`

### 2.2 Verificar Estructura

Después de crear, deberías ver:

```
aletza_db
├── usuarios
├── perfiles_voz
├── sesiones
├── mensajes
└── logs_ejecucion
```

---

## 📥 Paso 3: Inserción de Datos (Documentos)

### 3.1 Insertar Documentos en Colección `usuarios`

**Método: Insert Multiple Documents**

1. Hacer clic en la colección `usuarios`
2. Hacer clic en **"Insert Document"**
3. Seleccionar **"Insert Multiple Documents"**
4. Copiar y pegar el siguiente JSON:

```json
[
    {
        "_id": 1,
        "username": "carlos_dev",
        "email": "carlos.mendoza@email.com",
        "pais": "Perú",
        "ciudad": "Lima",
        "activo": true,
        "creado_en": { "$date": "2024-01-15T10:30:00Z" }
    },
    {
        "_id": 2,
        "username": "maria_lopez",
        "email": "maria.lopez@email.com",
        "pais": "México",
        "ciudad": "CDMX",
        "activo": true,
        "creado_en": { "$date": "2024-01-15T14:00:00Z" }
    },
    {
        "_id": 3,
        "username": "juan_torres",
        "email": "juan.torres@email.com",
        "pais": "Argentina",
        "ciudad": "Buenos Aires",
        "activo": true,
        "creado_en": { "$date": "2024-01-16T09:15:00Z" }
    },
    {
        "_id": 4,
        "username": "ana_rodriguez",
        "email": "ana.rodriguez@email.com",
        "pais": "Colombia",
        "ciudad": "Bogotá",
        "activo": true,
        "creado_en": { "$date": "2024-01-16T18:20:00Z" }
    },
    {
        "_id": 5,
        "username": "luis_fernandez",
        "email": "luis.fernandez@email.com",
        "pais": "España",
        "ciudad": "Madrid",
        "activo": true,
        "creado_en": { "$date": "2024-01-17T08:00:00Z" }
    },
    {
        "_id": 6,
        "username": "sofia_ramirez",
        "email": "sofia.ramirez@email.com",
        "pais": "Chile",
        "ciudad": "Santiago",
        "activo": true,
        "creado_en": { "$date": "2024-01-17T20:00:00Z" }
    },
    {
        "_id": 7,
        "username": "pedro_gomez",
        "email": "pedro.gomez@email.com",
        "pais": "Argentina",
        "ciudad": "Córdoba",
        "activo": true,
        "creado_en": { "$date": "2024-01-18T11:00:00Z" }
    },
    {
        "_id": 8,
        "username": "laura_martinez",
        "email": "laura.martinez@email.com",
        "pais": "Colombia",
        "ciudad": "Medellín",
        "activo": true,
        "creado_en": { "$date": "2024-01-18T15:30:00Z" }
    },
    {
        "_id": 9,
        "username": "diego_silva",
        "email": "diego.silva@email.com",
        "pais": "Perú",
        "ciudad": "Arequipa",
        "activo": true,
        "creado_en": { "$date": "2024-01-19T07:30:00Z" }
    },
    {
        "_id": 10,
        "username": "carmen_vega",
        "email": "carmen.vega@email.com",
        "pais": "México",
        "ciudad": "Guadalajara",
        "activo": true,
        "creado_en": { "$date": "2024-01-19T22:00:00Z" }
    }
]
```

5. Hacer clic en **"Insert"**

### 3.2 Insertar Documentos en Colección `perfiles_voz`

```json
[
    {
        "_id": 1,
        "usuario_id": 1,
        "palabra_magica": "aletza",
        "vector_voz": [0.12, 0.45, 0.78, 0.23],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-15T10:35:00Z" }
    },
    {
        "_id": 2,
        "usuario_id": 2,
        "palabra_magica": "aletza",
        "vector_voz": [0.34, 0.56, 0.12, 0.89],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-15T14:05:00Z" }
    },
    {
        "_id": 3,
        "usuario_id": 3,
        "palabra_magica": "aletza",
        "vector_voz": [0.56, 0.78, 0.34, 0.12],
        "muestras_tomadas": 8,
        "estado": "incompleto",
        "creado_en": { "$date": "2024-01-16T09:20:00Z" }
    },
    {
        "_id": 4,
        "usuario_id": 4,
        "palabra_magica": "aletza",
        "vector_voz": [0.78, 0.12, 0.56, 0.34],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-16T18:25:00Z" }
    },
    {
        "_id": 5,
        "usuario_id": 5,
        "palabra_magica": "aletza",
        "vector_voz": [0.23, 0.67, 0.89, 0.45],
        "muestras_tomadas": 5,
        "estado": "incompleto",
        "creado_en": { "$date": "2024-01-17T08:05:00Z" }
    },
    {
        "_id": 6,
        "usuario_id": 6,
        "palabra_magica": "aletza",
        "vector_voz": [0.45, 0.89, 0.23, 0.67],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-17T20:05:00Z" }
    },
    {
        "_id": 7,
        "usuario_id": 7,
        "palabra_magica": "aletza",
        "vector_voz": [0.67, 0.34, 0.45, 0.78],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-18T11:05:00Z" }
    },
    {
        "_id": 8,
        "usuario_id": 8,
        "palabra_magica": "aletza",
        "vector_voz": [0.89, 0.23, 0.67, 0.56],
        "muestras_tomadas": 3,
        "estado": "incompleto",
        "creado_en": { "$date": "2024-01-18T15:35:00Z" }
    },
    {
        "_id": 9,
        "usuario_id": 9,
        "palabra_magica": "aletza",
        "vector_voz": [0.34, 0.78, 0.12, 0.45],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-19T07:35:00Z" }
    },
    {
        "_id": 10,
        "usuario_id": 10,
        "palabra_magica": "aletza",
        "vector_voz": [0.56, 0.12, 0.78, 0.34],
        "muestras_tomadas": 10,
        "estado": "completo",
        "creado_en": { "$date": "2024-01-19T22:05:00Z" }
    }
]
```

### 3.3 Insertar Documentos en Colección `sesiones`

```json
[
    {
        "_id": 1,
        "usuario_id": 1,
        "token": "tok_abc123",
        "canal": "telegram",
        "iniciada_en": { "$date": "2024-01-15T10:30:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T11:00:00Z" },
        "expira_en": { "$date": "2024-01-15T11:30:00Z" },
        "cerrada": true
    },
    {
        "_id": 2,
        "usuario_id": 1,
        "token": "tok_def456",
        "canal": "web",
        "iniciada_en": { "$date": "2024-01-16T09:15:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T09:45:00Z" },
        "expira_en": { "$date": "2024-01-16T10:15:00Z" },
        "cerrada": true
    },
    {
        "_id": 3,
        "usuario_id": 2,
        "token": "tok_ghi789",
        "canal": "telegram",
        "iniciada_en": { "$date": "2024-01-15T14:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T14:30:00Z" },
        "expira_en": { "$date": "2024-01-15T15:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 4,
        "usuario_id": 2,
        "token": "tok_jkl012",
        "canal": "app",
        "iniciada_en": { "$date": "2024-01-16T18:20:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T18:50:00Z" },
        "expira_en": { "$date": "2024-01-16T19:20:00Z" },
        "cerrada": true
    },
    {
        "_id": 5,
        "usuario_id": 3,
        "token": "tok_mno345",
        "canal": "web",
        "iniciada_en": { "$date": "2024-01-15T08:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T08:30:00Z" },
        "expira_en": { "$date": "2024-01-15T09:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 6,
        "usuario_id": 4,
        "token": "tok_pqr678",
        "canal": "telegram",
        "iniciada_en": { "$date": "2024-01-16T20:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T20:30:00Z" },
        "expira_en": { "$date": "2024-01-16T21:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 7,
        "usuario_id": 5,
        "token": "tok_stu901",
        "canal": "app",
        "iniciada_en": { "$date": "2024-01-15T12:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T12:30:00Z" },
        "expira_en": { "$date": "2024-01-15T13:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 8,
        "usuario_id": 6,
        "token": "tok_vwx234",
        "canal": "telegram",
        "iniciada_en": { "$date": "2024-01-16T11:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T11:30:00Z" },
        "expira_en": { "$date": "2024-01-16T12:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 9,
        "usuario_id": 7,
        "token": "tok_yz5678",
        "canal": "web",
        "iniciada_en": { "$date": "2024-01-15T16:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T16:30:00Z" },
        "expira_en": { "$date": "2024-01-15T17:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 10,
        "usuario_id": 8,
        "token": "tok_abc890",
        "canal": "app",
        "iniciada_en": { "$date": "2024-01-16T07:30:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T08:00:00Z" },
        "expira_en": { "$date": "2024-01-16T08:30:00Z" },
        "cerrada": true
    },
    {
        "_id": 11,
        "usuario_id": 9,
        "token": "tok_def123",
        "canal": "telegram",
        "iniciada_en": { "$date": "2024-01-15T22:00:00Z" },
        "ultima_actividad": { "$date": "2024-01-15T22:30:00Z" },
        "expira_en": { "$date": "2024-01-15T23:00:00Z" },
        "cerrada": true
    },
    {
        "_id": 12,
        "usuario_id": 10,
        "token": "tok_ghi456",
        "canal": "web",
        "iniciada_en": { "$date": "2024-01-16T15:30:00Z" },
        "ultima_actividad": { "$date": "2024-01-16T16:00:00Z" },
        "expira_en": { "$date": "2024-01-16T16:30:00Z" },
        "cerrada": true
    }
]
```

### 3.4 Insertar Documentos en Colección `mensajes`

```json
[
    {
        "_id": 1,
        "sesion_id": 1,
        "remitente_id": 1,
        "contenido": "Hola Aletza, ¿cómo estás?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "saludo",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-15T10:30:15Z" }
    },
    {
        "_id": 2,
        "sesion_id": 1,
        "remitente_id": 1,
        "contenido": "Quiero saber el clima en Lima",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "clima",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T10:31:00Z" }
    },
    {
        "_id": 3,
        "sesion_id": 1,
        "remitente_id": 1,
        "contenido": "Gracias por la información",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "agradecimiento",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-15T10:32:00Z" }
    },
    {
        "_id": 4,
        "sesion_id": 2,
        "remitente_id": 1,
        "contenido": "¿Puedes ayudarme con soporte técnico?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "soporte",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-16T09:16:00Z" }
    },
    {
        "_id": 5,
        "sesion_id": 2,
        "remitente_id": 1,
        "contenido": "Mi cuenta no carga correctamente",
        "modalidad": "audio",
        "metadata_ia": {
            "intencion": "soporte",
            "sentimiento": "negativo",
            "transcripcion": "mi cuenta no carga correctamente"
        },
        "enviado_en": { "$date": "2024-01-16T09:18:00Z" }
    },
    {
        "_id": 6,
        "sesion_id": 3,
        "remitente_id": 2,
        "contenido": "aletza",
        "modalidad": "audio",
        "metadata_ia": {
            "intencion": "autenticacion",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T14:00:10Z" }
    },
    {
        "_id": 7,
        "sesion_id": 3,
        "remitente_id": 2,
        "contenido": "Reproduce mi playlist favorita",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "musica",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-15T14:01:00Z" }
    },
    {
        "_id": 8,
        "sesion_id": 3,
        "remitente_id": 2,
        "contenido": "¿Qué canciones tiene?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "consulta",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T14:02:30Z" }
    },
    {
        "_id": 9,
        "sesion_id": 4,
        "remitente_id": 2,
        "contenido": "Dame recomendaciones de películas",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "recomendacion",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-16T18:21:00Z" }
    },
    {
        "_id": 10,
        "sesion_id": 5,
        "remitente_id": 3,
        "contenido": "¿Cuál es el horario de atención?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "informacion",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T08:01:00Z" }
    },
    {
        "_id": 11,
        "sesion_id": 6,
        "remitente_id": 4,
        "contenido": "Hola",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "saludo",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-16T20:01:00Z" }
    },
    {
        "_id": 12,
        "sesion_id": 6,
        "remitente_id": 4,
        "contenido": "Quiero aprender SQL",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "educacion",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-16T20:05:00Z" }
    },
    {
        "_id": 13,
        "sesion_id": 7,
        "remitente_id": 5,
        "contenido": "¿Cómo configuro mi cuenta?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "soporte",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T12:05:00Z" }
    },
    {
        "_id": 14,
        "sesion_id": 8,
        "remitente_id": 6,
        "contenido": "¡Excelente servicio!",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "feedback",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-16T11:05:00Z" }
    },
    {
        "_id": 15,
        "sesion_id": 9,
        "remitente_id": 7,
        "contenido": "Necesito cambiar mi contraseña",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "soporte",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T16:02:00Z" }
    },
    {
        "_id": 16,
        "sesion_id": 10,
        "remitente_id": 8,
        "contenido": "Buenos días",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "saludo",
            "sentimiento": "positivo"
        },
        "enviado_en": { "$date": "2024-01-16T07:32:00Z" }
    },
    {
        "_id": 17,
        "sesion_id": 11,
        "remitente_id": 9,
        "contenido": "Crear recordatorio para mañana",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "recordatorio",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-15T22:05:00Z" }
    },
    {
        "_id": 18,
        "sesion_id": 12,
        "remitente_id": 10,
        "contenido": "¿Cómo está el clima en Barcelona?",
        "modalidad": "texto",
        "metadata_ia": {
            "intencion": "clima",
            "sentimiento": "neutral"
        },
        "enviado_en": { "$date": "2024-01-16T15:32:00Z" }
    }
]
```

