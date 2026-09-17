# 🎙️ Documento de Especificación de Proyecto: Plataforma SaaS de Agentes de Voz con IA

## 1. Resumen Ejecutivo

El presente proyecto tiene como objetivo el diseño, desarrollo y despliegue de una plataforma SaaS (Software as a Service) basada en agentes conversacionales de voz potenciados por Inteligencia Artificial.

La solución permitirá a empresas y organizaciones automatizar interacciones telefónicas (atención al cliente, soporte, reservas y prospección comercial) mediante agentes con lenguaje natural, alta expresividad y mínima latencia. El sistema se estructurará bajo una arquitectura multi-tenant, garantizando el aislamiento de datos, la escalabilidad y la personalización independiente por cliente.

## 2. Arquitectura de Alto Nivel

### 2.1 Modelo Multi-tenant

- **Aislamiento de Datos:** Separación lógica estricta mediante identificadores de inquilino (`tenant_id`) en cada entidad del modelo de datos, reforzada con políticas de acceso a nivel de fila (Row Level Security - RLS).
- **Configuración Independiente:** Cada cliente puede configurar la personalidad de su agente, base de conocimientos, voz, números telefónicos asignados y reglas de negocio.
- **Escalabilidad y Seguridad:** Control de acceso basado en roles (RBAC) para miembros de cada organización cliente.

### 2.2 Pipeline de Procesamiento de Voz en Tiempo Real

El ciclo de interacción sigue un flujo optimizado para mantener una latencia conversacional inferior a 1 segundo:

```text
[Usuario Habla]
       │
       ▼
1. Speech-to-Text (STT) ──► Conversión de voz a texto en streaming de baja latencia
       │
       ▼
2. LLM Engine           ──► Razonamiento, aplicación de contexto/RAG y generación de respuesta
       │
       ▼
3. Text-to-Speech (TTS) ──► Síntesis de voz hiperrealista con modulación natural
       │
       ▼
[Respuesta de Voz al Usuario]
```

## 3. Stack Tecnológico

| Componente                     | Tecnología            | Responsabilidad Principal                                                                                                                           |
| :----------------------------- | :-------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Backend API**                | Python + FastAPI      | Núcleo de la lógica de negocio, integración de webhooks, gestión de sesiones, comunicación segura con la base de datos y orquestación de servicios. |
| **Orquestador de Voz IA**      | Vapi                  | Gestión del pipeline de voz en tiempo real (STT + LLM + TTS), control de latencia, detección de interrupciones y manejo de turnos conversacionales. |
| **Telefonía y Comunicaciones** | Twilio                | Provisión y gestión de numeración telefónica, enrutamiento de llamadas entrantes y salientes vía troncal SIP/WebRTC.                                |
| **Frontend Dashboard**         | React                 | Panel de administración y autoservicio para el cliente: configuración de agentes, visualización de métricas, historial de llamadas y analítica.     |
| **Capa de Datos y Auth**       | Supabase (PostgreSQL) | Gestión de autenticación de usuarios, base de datos relacional con RLS para aislamiento multi-tenant y almacenamiento de registros/audios.          |

## 4. Módulos y Funcionalidades Principales

### 🏢 Gestión de Inquilinos y Usuarios (Multi-tenant):

- Registro corporativo y gestión de perfiles.
- Asignación de roles y permisos por equipo.

### 🤖 Panel de Configuración del Agente de Voz:

- Definición de _system prompts_, personalidad e instrucciones específicas.
- Catálogo de voces sintéticas e idiomas.
- Integración de fuentes de conocimiento (FAQ, documentos, endpoints externos).

### 📞 Módulo de Telefonía:

- Vinculación de números telefónicos de Twilio a agentes específicos.
- Configuración de reglas de desvío, horarios de atención y transferencias a operadores humanos.

### 📊 Registro, Transcripción y Analítica:

- Almacenamiento y consulta de transcripciones completas de llamadas.
- Registro de métricas operativas (duración de llamadas, tasas de resolución, latencia media).

## 5. Mi propuesta

Para el SaaS de agentes de voz con IA, mi propuesta es utilizar un modelo multi-tenant con base de datos y schema compartidos, utilizando tenant_id en las tablas y RLS en PostgreSQL para aislar los datos de cada cliente.

La principal razón es que, si vamos a tener muchos clientes, crear una base de datos independiente para cada uno desde el principio aumentaría la complejidad de mantenimiento, migraciones, backups y despliegues. Con un modelo compartido, todos utilizan la misma estructura, pero cada registro pertenece a un tenant_id, y RLS garantiza que un cliente no pueda acceder a los datos de otro.

A nivel de seguridad, lo combinaría con JWT y RBAC. El JWT identifica al usuario, su tenant y su rol; RBAC controla qué acciones puede realizar según su rol, mientras que RLS garantiza el aislamiento de los datos. De esta forma, FastAPI gestiona la autenticación y autorización, y PostgreSQL añade una segunda capa de seguridad.
Además, dejaría la arquitectura preparada para que, en el futuro, un cliente Enterprise que necesite un mayor nivel de aislamiento pueda utilizar una base de datos dedicada.
Con este enfoque buscamos un equilibrio entre seguridad, escalabilidad y facilidad de mantenimiento, manteniendo la posibilidad de adaptar el nivel de aislamiento según las necesidades de cada cliente.

## 6. Próximos Hitos de Desarrollo

- **Fase 1:** Definición del esquema de datos multi-tenant en Supabase y configuración de políticas RLS.
- **Fase 2:** Implementación del backend en FastAPI y configuración de webhooks de integración con Vapi.
- **Fase 3:** Conexión de Twilio con Vapi para pruebas de llamadas entrantes (inbound) y salientes (outbound).
- **Fase 4:** Desarrollo del panel de control en React y conexión con los endpoints del backend.
- **Fase 5:** Pruebas de carga, optimización de latencia y despliegue del Producto Mínimo Viable (MVP).
