# iam-service

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Kevin Andrey Culma (Arquitecto) | Equipo: Arquitectura.

| Campo | Valor |
|---|---|
| Módulo | M1 |
| App repo | `design-software-iam-service` |
| DB repo | `design-software-iam-db` |
| MFE repo | `design-software-iam-mfe` |
| Stack | Java / Spring Boot / Maven |
| Deployables | iam-api |
| Estado observado | API implementada; ampliación R3 y ajuste del consumidor MFE publicados en PR abiertas, sin fusionar al corte del 21 de octubre. |

## Responsabilidad

Identidad, autenticación, sesiones, RBAC, JWT/JWKS y auditoría de login.

## Persistencia

- Esquemas: identity, identity_audit, rbac, rbac_catalog, session
- Tablas: identity.outbox, identity.user, identity_audit.audit_login, rbac.role, rbac.role_feature, rbac.user_role, rbac.user_scope_override, rbac_catalog.feature, rbac_catalog.module, session.password_reset_request, session.refresh_token

## API detectada

| Metodo | Ruta | Fuente |
|---|---|---|
| GET | `/api/v1/auth/.well-known/jwks.json` | `.\design-software-iam-service\iam-adapters\src\main\java\co\sena\iam\adapter\in\rest\JwksController.java` |
| GET | `/health` | `.\design-software-iam-service\iam-adapters\src\main\java\co\sena\iam\adapter\in\rest\HealthController.java` |
| GET | `/ready` | `.\design-software-iam-service\iam-adapters\src\main\java\co\sena\iam\adapter\in\rest\HealthController.java` |
| GET | `/actuator/prometheus` (interno, rama R3) | `iam-api/src/main/resources/application.yml` y Micrometer Prometheus, entrega `7c837e0` |

El contrato completo está en [OpenAPI IAM](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/contracts/openapi/iam.yaml).
El consumidor R3 usa `POST /api/v1/auth/login` con `email/password` y recibe
`TokenResponse` plano; el [contrato narrativo](components/iam-api/contract.md)
describe la correlación y la entrega de sesión al host.

## Configuración

- `IAM_DB_DSN`
- `IAM_DB_PASSWORD`
- `IAM_DB_USERNAME`
- `IAM_EVENTS_EXCHANGE`
- `IAM_JWT_ISSUER`
- `IAM_JWT_KEY_ID`
- `IAM_JWT_PREVIOUS_KEY_ID`
- `IAM_JWT_PREVIOUS_PUBLIC_KEY`
- `IAM_JWT_PRIVATE_KEY`
- `IAM_OUTBOX_RELAY_BATCH_SIZE`
- `IAM_OUTBOX_RELAY_INITIAL_DELAY_MS`
- `IAM_OUTBOX_RELAY_POLL_INTERVAL_MS`

La configuración de broker, OpenTelemetry y los procedimientos están en el
[runbook](runbook.md). La conexión PostgreSQL se configura con `IAM_DB_*`;
el servicio valida el esquema previamente migrado y no aplica DDL al arrancar.

## Entrega del plan de mejoramiento — corte 2026-10-21

| Proyecto | Rama y commit revisados | PR hacia `develop` | Resultado al corte |
|---|---|---|---|
| Service | `r3-observability-delivery-dev`, `7c837e0824263cef0aef89f807731adca839f25e` | [#43](https://github.com/code-sena/design-software-iam-service/pull/43), abierta | CI aprobada: 186 casos; 55 IT omitidas en CI. |
| DB | `r3-iam-db-integration-dev`, `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8` | [#79](https://github.com/code-sena/design-software-iam-db/pull/79), abierta | CI de compatibilidad aprobada; sin DDL nuevo. |
| MFE | `r3-iam-mfe-integration-dev`, `e73e844bebebb64b99bfce24a908926f0e4cf76b` | [#10](https://github.com/code-sena/design-software-iam-mfe/pull/10), abierta | 46 pruebas locales; CI detenida por credencial de dependencias privadas ausente. |

La ampliación del Service añade validación/devolución de `X-Correlation-Id`,
restauración del MDC, prioridad del `trace_id` activo para errores y métricas
HTTP Prometheus con etiqueta `service="iam-service"` e histogramas. El agente
OpenTelemetry del Dockerfile, los logs JSON y las capacidades de negocio ya
existían; sus regresiones no son funcionalidades nuevas de este plan.

Las ejecuciones locales conservadas aprobaron 186 casos de servicio y 54 de
integración PostgreSQL, con una integración RabbitMQ omitida. Se aplicaron 37
changesets oficiales a una base aislada; no se migró la base compartida. El
smoke del adaptador MFE mediante proxy alcanzó API/PostgreSQL, sin montar el
web-shell en un navegador. Quedan sin acreditar la imagen Docker nueva, el
usuario de aplicación real, consumidores, trazas distribuidas, scrape, Loki y
alertas IAM. No se ejecutaron pruebas nuevas para esta actualización documental.

El [expediente IAM](../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md)
y su [matriz de evidencias](../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md)
relacionan actividades, pruebas, archivos, autorías y límites; no constituyen
acta de aceptación ni cierre del plan.

## Notas

La implementación existente publica eventos IAM por outbox/RabbitMQ cuando el
broker está configurado y expone JWKS para validación de JWT por otros servicios.
La entrega R3 no acredita publicación/consumo RabbitMQ ni fusión de las PR.
