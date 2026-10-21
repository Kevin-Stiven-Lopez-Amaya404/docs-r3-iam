# Arquitectura del Sistema — SENA Gestión de Horarios

> Fase: 03-Design | Agente: A06 | Estado: 🟡 Revisión
> Fecha: 2026-10-21
> Prerequisitos: [pattern-guide.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/pattern-guide.md) · [discovery-brief.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/discovery-brief.md)

## Resumen ejecutivo

El sistema de Gestión de Horarios SENA es una plataforma de microservicios con
**10 servicios backend**, incluida la separación de Notification registrada en
[ADR-005](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-005-microservices-topology-and-transversal-capabilities.md).
Combina REST síncrono y eventos asíncronos. Cada servicio conserva propiedad
lógica de sus datos; en desarrollo se utiliza PostgreSQL compartido con
esquemas por módulo. Los diagramas de despliegue objetivo no acreditan que
todas las réplicas, bases o componentes estén desplegados.

| Dimensión | Decisión |
|-----------|----------|
| Estilo arquitectural | Microservicios con DDD + Hexagonal por servicio |
| Comunicación sync | REST/HTTP con circuit breaker y timeout 5 s |
| Comunicación async | RabbitMQ en el perfil IAM existente; diseño de eventos en ADR-001. La entrega R3 no valida consumidores. |
| Persistencia | Propiedad de datos por servicio; esquemas por módulo en PostgreSQL compartido de desarrollo |
| Autenticación | JWT Bearer (emitido por `iam-service`) |
| Almacenamiento de objetos | Object Storage S3/MinIO (via `document-service`) |
| Caché | Redis por consumidor, TTL según volatilidad del dato |
| Observabilidad | OpenTelemetry + logs estructurados JSON + métricas RED |

### Corte verificable de la entrega IAM

La ampliación de correlación/Micrometer de Service está en
`r3-observability-delivery-dev`, commit `7c837e0824263cef0aef89f807731adca839f25e`,
[PR #43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta.
Las PR [DB #79](https://github.com/code-sena/design-software-iam-db/pull/79) y
[MFE #10](https://github.com/code-sena/design-software-iam-mfe/pull/10) también
permanecían abiertas al 21 de octubre. El cambio DB comprueba compatibilidad
sin DDL nuevo; MFE corrige el consumidor del contrato de autenticación existente.

Se conservan 186 casos de Service y 54 IT PostgreSQL aprobados localmente, con
una IT de broker omitida, y 46 casos MFE. Un smoke del adaptador atravesó proxy
Vite, API y PostgreSQL; no montó el shell en un navegador. Service y DB tienen
CI aprobada; las 55 IT externas se omiten en CI de Service y la CI MFE falla
por falta de credencial de lectura de dependencias privadas. No se acreditan
imagen Docker nueva, usuario real de aplicación, consumidores ni ingesta de
observabilidad IAM. El [informe técnico](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md)
relaciona resultados históricos y límites; esta revisión no ejecutó pruebas.

---

## Visión de componentes — C4 Nivel 2 (Contenedores)

```
                           ┌─────────────────────────────┐
                           │         USUARIOS             │
                           │  Coordinador / Instructor /  │
                           │  Aprendiz / Administrador    │
                           └──────────────┬──────────────┘
                                          │ HTTPS
                                          ▼
                           ┌─────────────────────────────┐
                           │         API GATEWAY          │
                           │   Routing · Rate Limiting    │
                           │   SSL Termination · CORS     │
                           └──────────────┬──────────────┘
                                          │
              ┌───────────────────────────┼────────────────────────────┐
              │                           │                            │
              ▼                           ▼                            ▼
  ┌───────────────────┐      ┌───────────────────────┐    ┌───────────────────┐
  │   iam-service     │      │  scheduling-service    │    │  actors-service   │
  │  :8080 interno    │      │  :8005                 │    │  :8006            │
  │  JWT · Roles      │      │  Motor horarios        │    │  Instructores     │
  │  PostgreSQL       │      │  PostgreSQL             │    │  Aprendices       │
  └───────────────────┘      │  + Redis caché          │    │  PostgreSQL       │
                             └────────────┬────────────┘    └───────────────────┘
                                          │ sync
              ┌───────────────────────────┼──────────────────────────┐
              ▼                           ▼                           ▼
  ┌───────────────────┐      ┌────────────────────────┐  ┌────────────────────┐
  │ reference-data-   │      │  training-environment- │  │ academic-management│
  │ service :8002     │      │  service :8004          │  │ -service :8003     │
  │  Catálogos M2+M4  │      │  Ambientes M3           │  │  Programas M5+M6   │
  │  PostgreSQL+Redis │      │  PostgreSQL             │  │  PostgreSQL        │
  └───────────────────┘      └────────────────────────┘  └────────────────────┘

  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ BROKER DE MENSAJES ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─

  ┌───────────────────┐      ┌────────────────────────┐  ┌────────────────────┐
  │ monitoring-       │      │  document-service      │  │  audit-service     │
  │ service :8008     │      │  :8007                 │  │  :8009             │
  │  KPIs M9          │      │  PDFs · Object Storage │  │  Append-only log   │
  │  PostgreSQL       │      │  PostgreSQL + MinIO/S3 │  │  PostgreSQL        │
  └───────────────────┘      └────────────────────────┘  └────────────────────┘
```

La vista resume componentes de negocio; Notification es el décimo servicio
separado. Los puertos restantes son referencias de diseño: la convención y los
prefijos edge están en [ADR-007](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-007-edge-gateway-and-port-convention.md).
El smoke IAM usó loopback `18080`, no esta topología completa.

---

## Flujos principales

### Flujo 1 — Crear y publicar un horario

```
Coordinador
  │
  ├─▶ POST /api/v1/auth/login     [iam-service] → TokenResponse
  │
  ├─▶ GET /academic/fichas?status=active   [academic-management-service]
  │       └─ Retorna fichas activas del centro
  │
  ├─▶ GET /environments/available?...      [training-environment-service]
  │       └─ Retorna ambientes disponibles para franja horaria (< 300 ms)
  │
  ├─▶ GET /actors/instructors/available?... [actors-service]
  │       └─ Retorna instructores disponibles con competencia requerida (< 300 ms)
  │
  ├─▶ POST /schedules                      [scheduling-service]
  │       └─ Crea borrador; estado: BORRADOR
  │
  ├─▶ POST /schedules/{id}/sessions        [scheduling-service]
  │       └─ Agrega sesiones de clase al borrador
  │
  ├─▶ POST /schedules/{id}/validate        [scheduling-service]
  │       └─ Detecta conflictos → responde lista de conflictos o vacío
  │
  └─▶ POST /schedules/{id}/publish         [scheduling-service]
        ├─ Cambia estado a PUBLICADO
        └─ Publica evento: scheduling.schedule.published
                            │
                            ├─▶ [monitoring-service] consume → actualiza estado seguimiento ficha
                            ├─▶ [audit-service] consume → registra evento de auditoría
                            └─▶ [notification-service / notification-worker] → notifica a instructor/aprendiz
```

### Flujo 2 — Consultar horario (instructor)

```
Instructor
  │
  ├─▶ GET /api/v1/auth/me                  [iam-service] → datos del actor
  │
  └─▶ GET /schedules/instructor/{id}/week?week=2026-W25  [scheduling-service]
        └─ Retorna sesiones de clase del instructor para la semana
```

### Flujo 3 — Seguimiento de ficha

```
Coordinador / Instructor
  │
  ├─▶ POST /monitoring/sessions            [monitoring-service]
  │       └─ Registra sesión de seguimiento (asistencia, avance)
  │
  └─▶ GET /monitoring/fichas/{id}/kpi      [monitoring-service]
        └─ Retorna KPIs: % asistencia, % avance, alertas activas
```

---

## Contratos y responsabilidades de borde

| Servicio | Expone | Consume (sync) | Consume (async) |
|----------|--------|----------------|-----------------|
| `iam-service` | Autenticación, JWT/JWKS, usuarios/RBAC, sondas; Prometheus interno en R3 | Actors para el claim de ficha de aprendiz, según contrato existente | — |
| `reference-data-service` | Catálogos, jerarquía institucional | — | — |
| `academic-management-service` | Programas, competencias, fichas | `iam-service` (auth) | — |
| `training-environment-service` | Disponibilidad de ambientes | `iam-service` (auth) | — |
| `actors-service` | Disponibilidad de instructores, datos de aprendices | `iam-service` (auth) | — |
| `scheduling-service` | Motor de horarios, detección de conflictos | `actors-service`, `training-environment-service`, `academic-management-service` | — |
| `document-service` | Generación de PDFs, almacenamiento de archivos | `iam-service` (auth) | `academic-events`, `scheduling-events` |
| `monitoring-service` | KPIs, alertas, seguimiento | `iam-service` (auth) | `scheduling-events`, `academic-events` |
| `audit-service` | (sin API pública) | — | todos los topics (wildcard) |

IAM también publica mediante su relay existente cuando se configura RabbitMQ;
el recorrido de publicación y consumo no fue validado por esta entrega R3.

---

## Requisitos no funcionales

### Rendimiento

| Operación | Objetivo p95 | Justificación |
|-----------|-------------|---------------|
| `GET /environments/available` | < 300 ms | Operación crítica durante creación de horario |
| `GET /instructors/available` | < 300 ms | Operación crítica durante creación de horario |
| `POST /schedules/{id}/validate` | < 2 s | Validación de horario completo (20 sesiones) |
| Publicación de evento | < 100 ms al broker | Desacoplado de la respuesta HTTP al usuario |
| Consulta de horario (instructor/aprendiz) | < 500 ms | UX aceptable en consulta |

### Disponibilidad

| Servicio | SLO Disponibilidad | Horario de criticidad |
|----------|-------------------|----------------------|
| `iam-service` | 99.9 % | 24/7 |
| `scheduling-service` | 99.5 % | 06:00–22:00 hora Colombia |
| `reference-data-service` | 99.5 % | Horas laborales |
| Resto de servicios | 99 % | Horas laborales |

### Seguridad

- Las operaciones protegidas requieren `Authorization: Bearer <JWT>`; las
  excepciones públicas de login, refresh, recuperación, JWKS y sondas están en
  [OpenAPI IAM](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/contracts/openapi/iam.yaml). Prometheus es operacional
  interno y debe excluirse de la ruta pública.
- Validación de JWT en cada servicio (stateless); `iam-service` emite, cada servicio verifica
- HTTPS en todos los canales; HTTP Strict Transport Security habilitado en gateway
- No se almacenan datos sensibles en logs (ver [security-rules.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/security-rules.md))

### Observabilidad

- El estándar transversal anterior usa `X-Request-ID`/`request_id`. En IAM R3,
  el filtro valida/devuelve `X-Correlation-Id` y añade `correlation_id` al MDC,
  restaurándolo al terminar. Los errores priorizan `trace_id` activo y conservan
  `X-Request-Id`/UUID como fallback. Esos identificadores no sustituyen W3C.
- IAM ya tenía logs JSON y agente Java OpenTelemetry. Son la base reutilizada;
  no son funcionalidades nuevas del plan ni prueban ingesta de señales.
- R3 expone métricas HTTP en `/actuator/prometheus`, con contador, histograma y
  etiqueta `service="iam-service"`. El endpoint local probado no acredita scrape,
  dashboard, p95 observado o alerta con recuperación.
- El stack elegido es OpenTelemetry + LGTM ([ADR-008](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-008-observability-opentelemetry.md)).
  La propagación W3C desde un span real del host y las trazas HTTP/JDBC en Tempo
  siguen sin acreditarse para esta entrega IAM; no se fabrican `traceparent`.

---

## Decisiones de arquitectura — Índice de ADRs

| ADR | Decisión | Estado |
|-----|----------|--------|
| [ADR-001](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-001-message-broker.md) | Selección de broker de mensajes (RabbitMQ) | 🟡 PROPOSED |
| [ADR-002](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-002-scheduling-read-models.md) | Reducción de dependencias síncronas en scheduling-service (read models) | 🟡 PROPOSED |
| [ADR-003](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-003-object-storage.md) | Estrategia de almacenamiento de objetos en document-service (MinIO/S3) | 🟡 PROPOSED |
| [ADR-004](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-004-status-parametrization-and-audit-standard.md) | Parametrización de estados (status_category/status) y estándar de auditoría | 🟡 PROPOSED |

---

## Topología de despliegue objetivo

```
┌─────────────────────────────────────────────────────────┐
│                  AMBIENTE DE PRODUCCIÓN                  │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │iam-api   │  │sched-api │  │actors-api│  │...     │  │
│  │ x2       │  │ x2       │  │ x2       │  │        │  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│                                                         │
│  ┌────────────────────────────────────────────────────┐ │
│  │              Broker de mensajes (cluster)           │ │
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │iam-db    │  │sched-db  │  │actors-db │  ... (1 BD   │
│  │PostgreSQL│  │PostgreSQL│  │PostgreSQL│  por servicio)│
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                         │
│  ┌──────────┐  ┌──────────┐                            │
│  │Redis     │  │MinIO/S3  │                            │
│  │(caché)   │  │(objetos) │                            │
│  └──────────┘  └──────────┘                            │
└─────────────────────────────────────────────────────────┘
```

Promoción prevista: `develop` → `qa` → `staging` → `main`; ver
[convenciones Git](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md) y
[deployment.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/deployment.md). Las PR IAM abiertas no constituyen promoción
ni despliegue de esta topología.

---

## Riesgos arquitecturales

| Riesgo | Probabilidad | Mitigación |
|--------|-------------|------------|
| `scheduling-service` con 3 deps síncronas (excede límite) | Confirmado | ADR-002: introducir read models para reducir a ≤ 2 |
| Latencia acumulada en flujos con múltiples saltos de servicio | Media | Medir p95 end-to-end desde sprint 1; budget: 1.5 s total para creación de sesión |
| Split-brain si el broker de mensajes falla durante publicación | Media | Outbox pattern en `scheduling-service` para garantizar at-least-once delivery |
| Compatibilidad de esquemas y permisos en PostgreSQL compartido | Requiere control por versión | Aplicar changelog oficial antes del arranque; Service valida el esquema, sin automigración. El preflight DB no acredita acceso efectivo del usuario desplegado. |
