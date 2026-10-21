# Mapa de configuración runtime

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Kevin Andrey Culma (Arquitecto) | Equipo: Arquitectura.

> Inventario general del 2026-10-07, con revisión específica de IAM al 2026-10-21. No se vuelve a certificar la configuración de los demás servicios. No incluir secretos reales en Git.

| Repo | Variables detectadas |
|---|---|
| `design-software-academic-management-service` | ACADEMIC_COMPETENCY_ROUTING_KEY, ACADEMIC_EVENTS_EXCHANGE, ACADEMIC_FICHA_ROUTING_KEY, ACTORS_SERVICE_BASE_URL, DB_PASSWORD, DB_URL, DB_USER, IAM_JWKS_URI, OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_TRACES_SAMPLING, RABBITMQ_HOST, RABBITMQ_PASSWORD, RABBITMQ_PORT, RABBITMQ_USERNAME, REFERENCE_DATA_BASE_URL |
| `design-software-actors-service` | ACADEMIC_MANAGEMENT_BASE_URL, ACTORS_DB_PASSWORD, ACTORS_DB_URL, ACTORS_DB_USER, IAM_JWKS_URI, IAM_JWT_ISSUER, KAFKA_BOOTSTRAP_SERVERS, OTEL_EXPORTER_OTLP_ENDPOINT |
| `design-software-audit-service` | PORT |
| `design-software-docker-infra` | APP_DB_PASSWORD, APP_DB_USER, BACKEND_REPOS_DIR, COMPOSE_PROJECT_NAME, DB_REPOS_DIR, EDGE_DASHBOARD_PORT, EDGE_PORT, GF_AUTH_ANONYMOUS_ENABLED, GF_AUTH_ANONYMOUS_ORG_ROLE, GRAFANA_PORT, IAM_DB_DSN, IAM_DB_PASSWORD, IAM_DB_USERNAME, IAM_JWT_ISSUER, IAM_JWT_PRIVATE_KEY, MAILHOG_SMTP_PORT, MAILHOG_UI_PORT, NOTIFICATION_AMQP_URL, NOTIFICATION_DB_DSN, NOTIFICATION_SMTP_ADDR, OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_EXPORTER_OTLP_INSECURE, OTEL_EXPORTER_OTLP_PROTOCOL, OTEL_SERVICE_NAME, OTLP_GRPC_PORT, OTLP_HTTP_PORT, POSTGRES_DB, POSTGRES_PASSWORD, POSTGRES_PORT, POSTGRES_USER, RABBITMQ_DEFAULT_PASS, RABBITMQ_DEFAULT_USER, RABBITMQ_MGMT_PORT, RABBITMQ_PASSWORD, RABBITMQ_PORT, RABBITMQ_USER, SPRING_RABBITMQ_HOST, SPRING_RABBITMQ_PASSWORD, SPRING_RABBITMQ_USERNAME |
| `design-software-iam-service` | IAM_DB_DSN, IAM_DB_PASSWORD, IAM_DB_USERNAME, IAM_EVENTS_EXCHANGE, IAM_JWT_ISSUER, IAM_JWT_KEY_ID, IAM_JWT_PREVIOUS_KEY_ID, IAM_JWT_PREVIOUS_PUBLIC_KEY, IAM_JWT_PRIVATE_KEY, IAM_OUTBOX_RELAY_BATCH_SIZE, IAM_OUTBOX_RELAY_INITIAL_DELAY_MS, IAM_OUTBOX_RELAY_POLL_INTERVAL_MS |
| `design-software-iam-db` (preflight) | PGHOST, PGPORT, PGDATABASE, PGUSER, PGPASSWORD; el workflow usa además LIQUIBASE_COMMAND_URL, LIQUIBASE_COMMAND_USERNAME, LIQUIBASE_COMMAND_PASSWORD para su PostgreSQL efímero |
| `design-software-iam-mfe` | VITE_IAM_API_URL (base API para el adaptador), IAM_API_PROXY_TARGET (destino del proxy Vite de desarrollo) |
| `design-software-notification-service` | ID, NOTIFICATION_AMQP_URL, NOTIFICATION_DB_DSN, NOTIFICATION_DEPLOYMENT_ENVIRONMENT, NOTIFICATION_RECIPIENT_STUB_EMAIL, NOTIFICATION_SMTP_ADDR, NOTIFICATION_SMTP_FROM, OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_EXPORTER_OTLP_INSECURE, PORT, WORKER_HEALTH_PORT |
| `design-software-reference-data-service` | HU_REF_018_ADMIN_EMAIL, HU_REF_018_ADMIN_PASSWORD, HU_REF_018_IAM_BASE_URL, HU_REF_018_LEARNER_EMAIL, HU_REF_018_LEARNER_PASSWORD, IAM_JWKS_URL, IAM_JWT_ISSUER, MESSAGE_BROKER_URL, OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_EXPORTER_OTLP_INSECURE, PORT, REFERENCE_DATA_AMQP_URL, REFERENCE_DATA_DATABASE_URL, REFERENCE_DATA_TEST_DSN |
| `design-software-scheduling-service` | OTEL_EXPORTER_OTLP_ENDPOINT, OTEL_TRACES_ENABLED, OTEL_TRACES_SAMPLER_PROBABILITY, POSTGRES_DB, POSTGRES_PASSWORD, POSTGRES_USER, SCHEDULING_DB_PASSWORD, SCHEDULING_DB_URL, SCHEDULING_DB_USERNAME, SCHEDULING_JPA_DDL_AUTO, SCHEDULING_JPA_SHOW_SQL, SCHEDULING_RABBIT_HEALTH_ENABLED, SCHEDULING_RABBIT_LISTENER_ENABLED, SERVER_PORT, SPRING_RABBITMQ_HOST, SPRING_RABBITMQ_PASSWORD, SPRING_RABBITMQ_PORT, SPRING_RABBITMQ_USERNAME |
| `design-software-training-environment-service` | APP_ENV, DATABASE_URL, ENV, ENVIRONMENT, ID, JWKS_JSON, JWKS_URL, JWT_PUBLIC_KEY, JWT_SECRET, PERF_DATE, PERF_SAMPLES, PORT, REQUIRE_POSTGRES_INTEGRATION, TEST_DATABASE_URL |

## Infra local

- `design-software-docker-infra` define la orquestación PostgreSQL, Liquibase, RabbitMQ, observabilidad, MailHog, Traefik y servicios conectados; esta tabla no certifica una instancia ejecutándose.
- PostgreSQL centraliza los esquemas por bounded context durante desarrollo local.
- RabbitMQ se usa para eventos/outbox y consumidores asincronos.

## Configuración IAM contrastada en la entrega

| Configuración | Responsable y significado | Evidencia/límite |
|---|---|---|
| `IAM_DB_DSN`, `IAM_DB_USERNAME`, `IAM_DB_PASSWORD` | Service; JDBC y credenciales de la aplicación por entorno | Service mantiene `ddl-auto: validate`. Los ensayos usaron propietario de base aislada; faltan credenciales/permisos del usuario real. |
| `VITE_IAM_API_URL` | MFE; base a la que se añaden `/auth/login`, `/auth/me` y recuperación | Configuración del bundle según entorno; no contiene credenciales. El proxy local probado no acredita la ruta edge de producción. |
| `IAM_API_PROXY_TARGET` | Servidor Vite; destino API del proxy de desarrollo | Smoke local Vite15101 → JAR18080. No equivale al montaje del shell. |
| `FRONTEND_DEPENDENCIES_READ_TOKEN` | Secreto GitHub Actions para lectura de repositorios frontend privados | Acceso a frontend-platform/design-system. CI MFE falló por ausencia antes de instalación/pruebas. No se entrega al navegador ni se publica su valor. |

Fuentes fijadas: [Service application.yml](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/iam-api/src/main/resources/application.yml), [MFE vite.config.ts](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/vite.config.ts), [MFE workflow](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/.github/workflows/ci.yml) y [DB preflight](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/scripts/verify-iam-integration.sql). Ver [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).
