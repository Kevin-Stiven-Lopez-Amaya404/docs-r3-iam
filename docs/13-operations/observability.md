# Observabilidad

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Operaciones

La operación del sistema **Horarios SENA** combina logs, métricas, trazas,
sondas y alertas. Hay componentes de datos y de aplicación; la capacidad
implementada debe distinguirse del recorrido comprobado en cada versión.
OpenTelemetry y LGTM (Collector, Tempo, Prometheus, Loki y Grafana) están
seleccionados por [ADR-008](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-008-observability-opentelemetry.md).

Este corte incorpora la entrega IAM Service
`7c837e0824263cef0aef89f807731adca839f25e`,
[PR #43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta
hacia `develop`. Sus resultados locales y la ampliación de métricas/correlación
no acreditan fusión, despliegue compartido o la operación completa del stack.
El [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md)
separa implementación, ejecuciones locales y CI. `monitoring` y `audit` son
además fuentes de observabilidad de negocio; sus evidencias tienen su propio
servicio, fecha y autoría.

## Fuentes de observabilidad

| Fuente | Naturaleza | Estado hoy |
|--------|------------|------------|
| PostgreSQL 16 (logs, `pg_stat_*`, `pg_isready`) | Infra de datos | ✅ Disponible hoy |
| Liquibase (`databasechangelog`, salida de `update`/`status`) | Trazabilidad de migraciones | ✅ Disponible hoy |
| IAM Service R3: `X-Correlation-Id` y MDC | Correlación de petición | Implementado y probado localmente; PR abierta |
| IAM Service R3: `/actuator/prometheus` | Contador e histograma HTTP | HTTP 200 y series locales comprobadas; scrape externo sin acreditar |
| IAM: agente OTel y logs JSON | Configuración de aplicación previa a R3 | Configuración existente; esta entrega no prueba ingesta Tempo/Loki |
| `monitoring-service` y su stack R3 | Observabilidad técnica y analítica de dominio | Evidencia propia en [expediente Monitoring](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/plan-mejoramiento/r3-observabilidad/README.md); no prueba IAM |
| `audit-service` (`audit_record`) | Registro de eventos de negocio | Consultar su documentación y evidencia específicas; fuera de esta validación IAM |

> **Distinción importante:** `monitoring` y `audit` son observabilidad **del negocio** (KPIs pedagógicos, historial de eventos), no reemplazan la observabilidad **técnica** (latencia, errores 5xx, saturación) de la plataforma. Ambas coexisten.

## Pilar 1 — Logs

### Datos y migraciones

- **Logs de PostgreSQL**: errores, `log_min_duration_statement` para consultas lentas, conexiones y checkpoints.
- **Salida de Liquibase**: cada `update`/`rollback`/`status` deja traza de qué changesets se aplicaron; la tabla `databasechangelog` (aislada por módulo) es el registro auditable de migraciones.

### Aplicación IAM

IAM ya tenía logs JSON por stdout y un agente Java OTel que aporta
`trace_id`/`span_id` al MDC cuando existe un span activo. R3 añade un filtro
que valida/devuelve `X-Correlation-Id`, usa `correlation_id` durante la petición
y restaura el contexto anterior incluso si falla el controlador. `ApiErrors`
prioriza el `trace_id` activo, luego `X-Request-Id` y UUID.

El estándar anterior nombra `request_id`/`X-Request-ID`; la ampliación IAM usa
`correlation_id`/`X-Correlation-Id`. Esa correspondencia no convierte ninguno de
ellos en W3C `traceparent`, ni modifica automáticamente el estándar de los demás
servicios. Loki es el destino elegido; su ingesta y la consulta de logs IAM
continúan sin evidencia en esta entrega. Retención y campos transversales:
[_template-observability.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-observability.md).

## Pilar 2 — Métricas

### Base de datos

Métricas útiles disponibles vía PostgreSQL sin capa de aplicación:

- Conexiones activas vs. límite del pool.
- Consultas lentas y locks (`pg_stat_activity`, `pg_locks`).
- Tamaño de BD y uso de disco (relevante para alertas de espacio).
- Éxito/fallo de la última ejecución de migraciones por módulo.

### Aplicación

La entrega IAM añade Micrometer Prometheus y exposición interna en
`GET /actuator/prometheus`, con `http_server_requests_seconds_count`,
`http_server_requests_seconds_bucket` y etiqueta `service="iam-service"`.
El smoke local conserva HTTP 200, contador e histograma; la suite automatizada
también comprueba estas series. RED por endpoint (Rate, Errors, Duration)
requiere scrape y consultas sobre muestras; disponer de buckets no acredita por
sí mismo un p95 observado ni una mejora de rendimiento.

Las métricas USE y los SLO por servicio requieren su propia medición.
Los KPIs pedagógicos de Monitoring son métricas de dominio y no sustituyen
latencia, tasa de errores o saturación de la plataforma.

## Pilar 3 — Trazas distribuidas

La configuración previa de IAM incluye agente Java para HTTP/JDBC, con destino
OTLP/HTTP `:4318` y protocolo `http/protobuf`. El host debe propagar el contexto
W3C de un span activo; el MFE admite fetch instrumentado y no fabrica
`traceparent`. `OTEL_SERVICE_NAME` configura la identidad OTel del entorno; no
cambia la etiqueta Micrometer derivada de `spring.application.name`.

El smoke del JAR R3 no cargó el agente. No conserva una traza IAM
navegador → shell → MFE → API → PostgreSQL consultada en Tempo, ni un recorrido
asíncrono por RabbitMQ/consumidor. Esas comprobaciones siguen pendientes para
IAM; las trazas de Monitoring no las sustituyen.

## Healthchecks

### Base de datos

```bash
# Disponibilidad de PostgreSQL en el contenedor
docker compose --env-file .env.develop exec postgres pg_isready -U <user> -d <db>
# Respuesta esperada: "accepting connections"

# Estado de migraciones de un módulo
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> status --verbose
```

`pg_isready` es la sonda mínima de vida de la BD y es la base del healthcheck del contenedor en Docker.

### Aplicación IAM

IAM expone `GET /health` (vida del proceso) y `GET /ready` (PostgreSQL y broker
solo cuando está configurado). Son rutas del `HealthController`, incluidas en
OpenAPI, distintas de Actuator. Las dos respondieron HTTP 200 en el smoke local.
No se debe deducir disponibilidad de RabbitMQ de ese resultado cuando estaba
sin configurar. Procedimientos y acceso interno:
[runbook IAM](../09-microservices/services/01-iam-service/runbook.md).

## Alertas

Marco de alertas (los umbrales son de referencia hasta fijar SLOs reales):

| Alerta | Condición (referencia) | Severidad | Alcance |
|--------|------------------------|-----------|------------|
| BD sin espacio | Disco > 85% | P1 | Referencia de operación de datos; disparo no probado por el plan IAM |
| PostgreSQL no responde | `pg_isready` falla | P0 | Referencia de operación de datos |
| Migración fallida | `update` de un módulo termina en error | P1 | Salida Liquibase disponible; alerta no probada por el plan IAM |
| Consultas lentas sostenidas | queries > umbral por N min | P2 | Umbral y muestras por entorno |
| Alta tasa de errores 5xx | >5% en ventana acordada | P1 | Métricas IAM disponibles; scrape y ciclo de alerta pendientes |
| Latencia alta | p95 > SLO | P2 | Requiere scrape, ventana y SLO del servicio |

Grafana forma parte del stack elegido. La entrega IAM no conserva pruebas de
disparo y recuperación ni valida el enrutamiento a un canal/guardia; los
umbrales anteriores son referencias, no evidencia de una alerta activa.
Ver [incident-management.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/incident-management.md).

## Puntos abiertos

- Scrape Prometheus, trazas HTTP/JDBC en Tempo, logs en Loki y dashboard IAM
  consultables con evidencia de fecha, versión y petición.
- Umbrales definitivos: dependen de los SLO por servicio (usar [_template-sla-slo-sli.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-sla-slo-sli.md)).
- Alerta IAM >5% con disparo y recuperación, y enlaces reales del entorno.
- Acceso del usuario de aplicación real, consumidores y ejecución completa desde
  web-shell; la validación local usó una base aislada y un driver HTTP.

## Resultados y procedencia de la validación IAM

Los registros conservados aprobaron 186 casos locales de Service y 54 IT contra
PostgreSQL, con una IT RabbitMQ omitida. El build JAR y las sondas aprobaron;
el build Docker falló por DNS antes de compilar. La CI de Service aprobó 186 y
omitió las 55 IT externas. DB tiene CI aprobada de migración/preflight; MFE
tiene 46 pruebas locales aprobadas y CI fallida antes de instalar dependencias
por credencial privada ausente. Son resultados históricos del plan, sin nueva
ejecución durante esta revisión documental. La
[matriz IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md)
relaciona código, pruebas, resultados, anexos y pendientes.

## Referencias

- [_template-observability.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-observability.md)
- [incident-management.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/incident-management.md)
- [backup-and-recovery.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/backup-and-recovery.md)
- [ADR-001 — Message broker](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-001-message-broker.md)
- [ADR-002 — Read models de scheduling](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-002-scheduling-read-models.md)
- [ADR-008 — OpenTelemetry y LGTM](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-008-observability-opentelemetry.md)
