# Informe: Arquitectura Multi-Tenant en un SaaS

## 1. ¿Qué es Multi-Tenant?

Una arquitectura **Multi-Tenant** permite que varios clientes utilicen
la misma aplicación SaaS, compartiendo parte de la infraestructura, pero
manteniendo sus datos y permisos separados.

Ejemplo:

``` text
             SaaS
              │
      ┌───────┼───────┐
      │       │       │
   Tenant A Tenant B Tenant C
```

El principal objetivo es conseguir **aislamiento, seguridad,
escalabilidad y eficiencia de costes**.

------------------------------------------------------------------------

## 2. Principales modelos

  --------------------------------------------------------------------------
  Modelo            Funcionamiento       Ventajas          Desventajas
  ----------------- -------------------- ----------------- -----------------
  **Pool**          Todos comparten BD y Barato y sencillo Mayor riesgo de
                    tablas                                 errores de
                                                           aislamiento

  **Bridge**        BD compartida,       Mayor aislamiento Más complejidad
                    schemas separados                      

  **Silo**          Cada tenant tiene su Máximo            Mayor coste y
                    propia               aislamiento       mantenimiento
                    BD/infraestructura                     

  **Híbrido**       Combina los          Flexible y        Mayor complejidad
                    anteriores           escalable         arquitectónica
  --------------------------------------------------------------------------

Para un SaaS en crecimiento, un modelo **híbrido** puede permitir
clientes pequeños en Pool y clientes Enterprise en Silo.

------------------------------------------------------------------------

## 3. Puntos fuertes

-   Menor coste de infraestructura.
-   Permite atender muchos clientes.
-   Aprovisionamiento rápido de nuevos tenants.
-   Facilita compartir servicios y código.
-   Escalabilidad horizontal.
-   Posibilidad de adaptar el nivel de aislamiento según el cliente.

------------------------------------------------------------------------

## 4. Puntos débiles

-   Mayor complejidad de seguridad.
-   Riesgo de fuga de datos entre tenants.
-   Un tenant puede consumir demasiados recursos.
-   La base de datos puede convertirse en cuello de botella.
-   Las migraciones pueden complicarse con muchos tenants.
-   Backups y restauraciones pueden ser más complejos.
-   Un fallo en un componente compartido puede afectar a muchos
    clientes.

------------------------------------------------------------------------

## 5. Principales problemas de seguridad

### Fuga de datos entre tenants

Un usuario del Tenant A nunca debe poder acceder a información del
Tenant B.

``` text
Tenant A → ❌ Datos Tenant B
```

### IDOR/BOLA

Un usuario modifica un ID:

``` text
/api/invoices/123
        ↓
/api/invoices/124
```

y consigue acceder a un recurso de otro tenant.

### Otros riesgos

-   Cachés compartidas incorrectamente.
-   Archivos almacenados sin aislamiento.
-   Webhooks asociados al tenant equivocado.
-   Jobs asíncronos sin contexto de tenant.
-   Permisos administrativos excesivos.
-   Backups accesibles incorrectamente.

------------------------------------------------------------------------

## 6. Principales cuellos de botella

### Base de datos

Es uno de los recursos compartidos más críticos.

Puede saturarse por:

-   demasiadas consultas;
-   consultas lentas;
-   demasiadas conexiones;
-   grandes cantidades de datos.

### Noisy Neighbor

Un tenant con un consumo excesivo puede perjudicar al resto.

``` text
Tenant A ─┐
Tenant B ─┼──► DB
Tenant C ─┘
             ↑
        Tenant C
        consume demasiado
```

### Otros posibles cuellos de botella

-   Redis/cache.
-   Colas y workers.
-   Almacenamiento.
-   APIs externas.
-   CPU y memoria.

------------------------------------------------------------------------

## 7. Soluciones principales

### Seguridad

-   Asociar cada recurso a un `tenant_id`.
-   Determinar el tenant desde la identidad autenticada.
-   No confiar únicamente en un `tenant_id` enviado por el cliente.
-   Utilizar RBAC/roles.
-   Aplicar autorización en cada operación.
-   Utilizar **Row Level Security (RLS)** cuando sea apropiado.
-   Realizar pruebas específicas de acceso entre tenants.

### Rendimiento

-   Índices adecuados.
-   Paginación.
-   Caché.
-   Rate limiting por tenant.
-   Límites de recursos.
-   Colas para operaciones pesadas.
-   Read replicas cuando sea necesario.
-   Separar tenants con cargas especialmente altas.

------------------------------------------------------------------------

## 8. Arquitectura recomendada

Una arquitectura inicial podría ser:

``` text
             Usuario
                │
                ▼
          API / Backend
                │
        ┌───────┴────────┐
        │ Tenant Context │
        └───────┬────────┘
                │
        Autenticación +
        autorización
                │
                ▼
           PostgreSQL
                │
          ┌─────┴─────┐
          │           │
        Pool        Silo
     clientes       clientes
      normales     Enterprise
```

De esta forma se puede comenzar con una infraestructura compartida y
aumentar el aislamiento cuando un cliente o sus requisitos lo
justifiquen.

------------------------------------------------------------------------

## 9. Conclusión

La arquitectura Multi-Tenant permite construir SaaS **más eficientes y
escalables**, pero introduce dos grandes retos:

**1. Aislar correctamente los datos.**\
**2. Evitar que un tenant perjudique al resto mediante un consumo
excesivo de recursos.**

La solución debe combinar:

``` text
Autenticación
      +
Autorización
      +
Tenant Isolation
      +
Rate Limiting
      +
Monitorización
      +
Backups
```

Para un SaaS moderno, una estrategia **híbrida Pool + Silo** puede
proporcionar un equilibrio entre **coste, seguridad, rendimiento y
escalabilidad**.
