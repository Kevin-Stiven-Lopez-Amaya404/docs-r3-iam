# iam-api

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Arquitectura

La API está implementada en Java/Spring Boot. La ampliación R3 de correlación y
métricas está en [Service PR #43](https://github.com/code-sena/design-software-iam-service/pull/43),
abierta hacia `develop` al corte del 21 de octubre, commit
`7c837e0824263cef0aef89f807731adca839f25e`. El ajuste MFE y el preflight DB
también permanecen en PR abiertas; consultar el [estado conjunto](../../README.md#entrega-del-plan-de-mejoramiento--corte-2026-10-21).

## Tipo de componente

- [x] `-api` — REST API sincrónica
- [ ] `-worker` — Consumidor de eventos / cola
- [ ] `-workflow` — Orquestación de pasos con compensaciones
- [ ] `-scheduler` — Tarea periódica
- [ ] `-notifier` — Envío de notificaciones salientes
- [ ] `-gateway` — Punto de entrada / proxy

## Responsabilidad

Expone los endpoints de autenticación (login, refresh, logout) y de administración de usuarios, roles, features y scopes; emite el JWT con features y scope pre-calculados que el resto de servicios consume para autorizar.

## Tecnologías

Tecnologías observadas en la entrega revisada:

| Capa | Tecnología | Versión |
|------|-----------|---------|
| Runtime | Java | 21 |
| Framework / build | Spring Boot / Maven Wrapper | Maven 3.9.10 |
| Base de datos | PostgreSQL; nombre y conexión configurables con `IAM_DB_DSN` | 16 en la validación local |
| Transporte de eventos | Broker AMQP (RabbitMQ, [ADR-001](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-001-message-broker.md)) | — |

La implementación existente prepara eventos de dominio en outbox y los publica
cuando el broker está configurado. La validación R3 omitió la IT del relay por
ausencia de broker; no acredita entrega a consumidores. Ver
[event-catalog.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/event-catalog.md).

## Variables de entorno requeridas

Variables reales del servicio; el [runbook](../../runbook.md#variables-de-entorno-requeridas)
es la referencia para obligatoriedad, broker y rotación de claves.

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `IAM_DB_DSN` | JDBC URL de la base previamente migrada | `jdbc:postgresql://postgres:5432/design-software-develop` |
| `IAM_DB_USERNAME` / `IAM_DB_PASSWORD` | Usuario y credencial de la base | Valores del entorno; no versionar credenciales |
| `IAM_JWT_PRIVATE_KEY` / `IAM_JWT_ISSUER` | Clave RSA y emisor de JWT | PKCS#8 / emisor acordado |
| `SPRING_RABBITMQ_HOST` y credenciales `SPRING_RABBITMQ_*` | Activar/configurar el broker del relay | `rabbitmq` en la red del entorno |
| `OTEL_EXPORTER_OTLP_ENDPOINT` / `OTEL_EXPORTER_OTLP_PROTOCOL` | Destino del agente OTel existente | `http://otel-collector:4318` / `http/protobuf` |
| `OTEL_SERVICE_NAME` | Identidad OTel acordada | `iam-service` recomendado en R3; validar el entorno |

El puerto configurado de la aplicación es `8080`; el smoke aislado usó `18080`.
El servicio conserva `ddl-auto: validate` y no aplica migraciones.

## Escalado, salud y observabilidad

- **Escalado:** sin estado en memoria (el estado de sesión vive en `refresh_token`); escala horizontalmente detrás de un balanceador. La verificación del JWT en servicios downstream no requiere llamar a `iam-api`.
- **Salud:** `/health` comprueba vida; `/ready` comprueba PostgreSQL y el broker
  solo cuando está configurado. Las tres comprobaciones locales de health,
  ready y Prometheus devolvieron HTTP 200 en el smoke conservado.
- **Correlación R3:** valida/devuelve `X-Correlation-Id`, restaura MDC y prioriza
  el `trace_id` activo para los errores. No fabrica contexto W3C.
- **Métricas R3:** `GET /actuator/prometheus` interno expone contadores e
  histogramas HTTP con `service="iam-service"`. El endpoint no acredita scrape.
- **Logs y trazas:** configuración JSON y agente OTel anteriores al plan. El
  smoke del JAR no cargó el agente; Tempo, Loki y alertas IAM no están acreditados
  en esta entrega. Ver [observabilidad](../../../../../13-operations/observability.md).

## Validación de la entrega

Los registros locales conservan 186 casos de servicio aprobados y 54 IT
PostgreSQL aprobadas, con una IT RabbitMQ omitida. La CI de Service aprobó 186
y omitió las 55 IT externas. No se produjo una imagen Docker nueva validada.
El [informe técnico](../../../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md)
relaciona estos resultados históricos, el smoke del adaptador MFE y sus límites;
no se ejecutaron pruebas nuevas para actualizar esta documentación.

## Contrato

Ver [contract.md](./contract.md)

## Referencias

- Modelo de datos: [../../data-model.md](../../data-model.md)
- Diseño RBAC (features, scopes, matriz de roles): [../../rbac-design.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/services/01-iam-service/rbac-design.md)
- Catálogo de eventos: [../../../../event-catalog.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/event-catalog.md)
