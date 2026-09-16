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

## 5. Próximos Hitos de Desarrollo

- **Fase 1:** Definición del esquema de datos multi-tenant en Supabase y configuración de políticas RLS.
- **Fase 2:** Implementación del backend en FastAPI y configuración de webhooks de integración con Vapi.
- **Fase 3:** Conexión de Twilio con Vapi para pruebas de llamadas entrantes (inbound) y salientes (outbound).
- **Fase 4:** Desarrollo del panel de control en React y conexión con los endpoints del backend.
- **Fase 5:** Pruebas de carga, optimización de latencia y despliegue del Producto Mínimo Viable (MVP).
