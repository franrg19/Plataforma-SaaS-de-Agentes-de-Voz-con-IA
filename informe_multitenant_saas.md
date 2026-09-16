# Informe: Arquitectura Multi-Tenant en un SaaS

## 1. ¿Qué es Multi-Tenant?

Una arquitectura **Multi-Tenant** permite que varios clientes utilicen
la misma aplicación SaaS, compartiendo parte de la infraestructura, pero
manteniendo sus datos y permisos separados.

Ejemplo:

```text
             SaaS
              │
      ┌───────┼───────┐
      │       │       │
   Tenant A Tenant B Tenant C
```

El principal objetivo es conseguir **aislamiento, seguridad,
escalabilidad y eficiencia de costes**.

### Ventajas

1. Eficiencia operativa: Permite gestionar una unica instancia de software en lugar de
   desplegar varias.
2. Reducción de costes: Optimiza el uso de recursos compartiendo infraestructura entre
   multiples clientes.
3. Mantenimiento simple: Las actualizaciones y mejoras se aplican una sola vez para
   todos los inquilinos.
4. Escablabilidad mejorada: Facilita el crecimiento tanto tecnico como organizativo.

---

## 2. Principales Modelos

### 1. Modelo Silo
Cada tenant tiene su propia infraestructura aislada:su base de datos, a veces su propio contenedor, su propia stack.

#### A favor:
- Aislamiento brutal. Un tenant no puede ni por accidente ver los datos de otro.
- Fácil de cumplir requisitos regulatorios estrictos.
- Un cliente pesado no afecta el rendimiento de los demás ("noisy neighbor").
#### En contra:
- Tu factura crece linealmente con cada cliente. 100 clientes = 100 bases de datos que pagar.
- Desplegar un cambio significa actualizarlo en N stacks.

---
### 2. Modelo POOL
Todos los tenants comparten la misma base de datos y la misma infraestructura. La separación es lógica, normalmente con una columna ```tenant_id``` en cada tabla.

#### A favor:
- Costo eficientísimo. Escalas a cientos de clientes sin multiplicar infraestructura.
- Un solo despliegue actualiza a todo el mundo.
#### En contra:
- El aislamiento depende 100% de tu código. Si a alguien se le olvida un ``WHERE tenant_id``, acabas de filtrar los datos de un cliente a otro. Es el bug más caro que existe en un SaaS.
- El "noisy neighbor" es real: un cliente con millones de registros puede ralentizar a todos.

### 3. Modelo Bridge
El híbrido: infraestructura compartida, pero datos separados por esquema o por base de datos dentro del mismo servidor.
#### A favor:
- Mejor aislamiento que Pool, más barato que Silo.
- Puedes hacer backup o migrar un solo tenant sin tocar a los demás.
#### En contra:
- La complejidad operativa. Las migraciones de esquema se vuelven un baile: tienes que correr cada migration contra N esquemas y rezar para que ninguno falle a la mitad.

## 4. Principales problemas de seguridad

### Fuga de datos entre tenants

Un usuario del Tenant A nunca debe poder acceder a información del
Tenant B.

```text
Tenant A → ❌ Datos Tenant B
```

### IDOR/BOLA

Un usuario modifica un ID:

```text
/api/invoices/123
        ↓
/api/invoices/124
```

y consigue acceder a un recurso de otro tenant.

### Otros riesgos

- Cachés compartidas incorrectamente.
- Archivos almacenados sin aislamiento.
- Webhooks asociados al tenant equivocado.
- Jobs asíncronos sin contexto de tenant.
- Permisos administrativos excesivos.
- Backups accesibles incorrectamente.

---

## 5. Principales cuellos de botella

### Base de datos

Es uno de los recursos compartidos más críticos.

Puede saturarse por:

- demasiadas consultas;
- consultas lentas;
- demasiadas conexiones;
- grandes cantidades de datos.

### Noisy Neighbor

Un tenant con un consumo excesivo puede perjudicar al resto.

```text
Tenant A ─┐
Tenant B ─┼──► DB
Tenant C ─┘
             ↑
        Tenant C
        consume demasiado
```

### Otros posibles cuellos de botella

- Redis/cache.
- Colas y workers.
- Almacenamiento.
- APIs externas.
- CPU y memoria.

---

## 6. Soluciones principales

### Seguridad

- Asociar cada recurso a un `tenant_id`.
- Determinar el tenant desde la identidad autenticada.
- No confiar únicamente en un `tenant_id` enviado por el cliente.
- Utilizar RBAC/roles.
- Aplicar autorización en cada operación.
- Utilizar **Row Level Security (RLS)** cuando sea apropiado.
- Realizar pruebas específicas de acceso entre tenants.

### Rendimiento

- Índices adecuados.
- Paginación.
- Caché.
- Rate limiting por tenant.
- Límites de recursos.
- Colas para operaciones pesadas.
- Read replicas cuando sea necesario.
- Separar tenants con cargas especialmente altas.

---

## 7. Consideraciones para Elegir una Arquitectura

### 1. Requisitos de aislamiento y seguridad:
¿Que nivel de separacion necesitan los datos?
### 2. Regulaciones y cumplimiento:
¿Que normativas deben de cumplirse?
### 3. Necesidades de personalización:
¿Cuanta adaptabilidad requiere cada inquilino?
### 4. Escala Esperada:
¿Cuantos inquilinos planea soportar y con que crecimiento?
### 5. Presupuesto disponible:
¿Que recursos financieros tienes para la infraestructura? 

---

## 8. Conclusión

La arquitectura Multi-Tenant permite construir SaaS **más eficientes y
escalables**, pero introduce dos grandes retos:

**1. Aislar correctamente los datos.**\
**2. Evitar que un tenant perjudique al resto mediante un consumo
excesivo de recursos.**


