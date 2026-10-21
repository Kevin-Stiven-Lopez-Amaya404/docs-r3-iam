# Setup local

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: DevOps

Cómo preparar la base de datos y los perfiles locales del proyecto. Los comandos de infraestructura son un procedimiento; los resultados efectivamente observados para la entrega IAM se identifican por separado. Complementa a [migration-strategy.md](../06-data/migration-strategy.md) y al [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

## Prerrequisitos

- Docker + Docker Compose.
- Repos `*-db` y servicios clonados en las rutas que resuelven `DB_REPOS_DIR` y `BACKEND_REPOS_DIR` de la infraestructura.
- Para reproducir la entrega IAM: Service `7c837e0824263cef0aef89f807731adca839f25e`, DB `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8` y MFE `e73e844bebebb64b99bfce24a908926f0e4cf76b`; sus PR siguen abiertas. Java 21 / Maven 3.9.10, Node 24 y acceso de lectura a las dependencias frontend privadas.

## Arquitectura local

Una **única base de datos PostgreSQL 16** en contenedor, y un servicio Liquibase por módulo (perfil `tooling`) que aplica el `changelog-master.yaml` del repo correspondiente. La configuración por ambiente vive en archivos de entorno: `.env.develop`, `.env.qa`, `.env.staging`, `.env.main`.

> **Nota de secretos:** los archivos `.env.*` **no deben contener contraseñas reales versionadas**. Usar `.env.example` como plantilla y mantener los valores reales fuera de git (ver [security-rules.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/security-rules.md)).

## Pasos

1. **Seleccionar ambiente y levantar Postgres.** Docker Compose lee `.env` por defecto, así que hay que pasar el archivo explícitamente:

   ```bash
   docker compose --env-file .env.develop up postgres -d
   ```

2. **Aplicar los changelogs de un módulo** (perfil `tooling`):

   ```bash
   docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> update
   ```

   Módulos disponibles: `liquibase-academic`, `liquibase-actors`, `liquibase-audit`, `liquibase-document`, `liquibase-iam`, `liquibase-monitoring`, `liquibase-reference`, `liquibase-scheduling`, `liquibase-training`.

3. **Verificar** el estado de las migraciones de un módulo:

   ```bash
   docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> status --verbose
   ```

## Rollback local

```bash
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> rollbackCount 1
```

Para volver a un release etiquetado: `rollback <tag>` (los tags se definen en `04_tcl/`; ver [migration-strategy.md](../06-data/migration-strategy.md)).

## Apagar y limpiar

```bash
docker compose --env-file .env.develop down          # detiene los contenedores
docker compose --env-file .env.develop down -v       # además borra el volumen de datos (reinicio limpio)
```

## Orden recomendado de aplicación

Aplicar primero `reference-data` (catálogos e institucional) e `iam` (identidad), y luego el resto de módulos que los referencian.

---

## Perfiles de aplicación (servicios de dominio)

Además de `tooling` (migraciones), la configuración de compose documenta perfiles por servicio de dominio. Un
servicio puede pertenecer a más de un perfil en la lista `profiles:` — así, `traefik` y `rabbitmq` se traen
solos al levantar el perfil del dominio que los necesita, sin tener que enumerar `--profile edge` o
`--profile broker` a mano.

| Perfil | Qué levanta | Build context (portable) |
|--------|-------------|---------------------------|
| `notification` | `notification-api` + `notification-worker` + `rabbitmq`/`mailhog` + `traefik` (edge) | `${BACKEND_REPOS_DIR:-../../backend}/design-software-notification-service` |
| `iam` | `iam-api` + `rabbitmq` (eventos `iam-events`) + `traefik` (edge) | `${BACKEND_REPOS_DIR:-../../backend}/design-software-iam-service` |
| `edge` | Solo `traefik`, si se necesita levantarlo sin ningún dominio | — |
| `observability` | Collector OTel + Tempo + Prometheus + Loki + Grafana (ADR-008) | — |

`BACKEND_REPOS_DIR` (default `../../backend`) apunta a la carpeta con los repos de backend clonados junto a
`docker-infra` — **nunca** a un worktree privado de una HU, para que el build sea reproducible para cualquiera
que clone el monorepo de repos tal como está documentado.

### Levantar iam-service local (HU-IAM-012)

```bash
# 1. Levantar PostgreSQL y aplicar el changelog antes del arranque de IAM.
docker compose --env-file .env.develop up postgres -d
docker compose --env-file .env.develop --profile tooling run --rm liquibase-iam validate
docker compose --env-file .env.develop --profile tooling run --rm liquibase-iam update

# 2. Configurar el usuario de aplicación y comprobar el esquema; después construir/levantar IAM.
docker compose --env-file .env.develop --profile iam up -d --build

# 3. Para validar exportación/ingesta E2E (ADR-008), agregar observability y recoger evidencia:
docker compose --env-file .env.develop --profile iam --profile observability up -d --build
```

> **Gotcha — contenedor stale:** levantá los servicios de aplicación con `up -d --build` y, **antes de reportar un bug de deploy** (un 404, un endpoint que "no existe"), **verificá el SHA de la imagen** que corre contra el que acabás de construir (`docker compose images` / `docker inspect`). Un contenedor viejo sin reconstruir ya dio falsos 404 que parecían bugs de código.

- **Acceso**: solo por el edge — `GET http://localhost:${EDGE_PORT:-18080}/iam/health` (ADR-007; el `-api` no
  se expone al host). El resto del contrato vive bajo `/iam/api/v1/...` (p. ej.
  `POST /iam/api/v1/auth/login`).
- **Env vars obligatorias** (sin default de secreto): `IAM_DB_DSN`, `IAM_DB_USERNAME`/`IAM_DB_PASSWORD` (usuario de
  aplicación, ver `create-app-user.sh`), `IAM_JWT_PRIVATE_KEY`, `IAM_JWT_ISSUER`. Ver
  [runbook.md del servicio](../09-microservices/services/01-iam-service/runbook.md) y
  [runtime-configuration-map.md](./runtime-configuration-map.md) para las variables; usar la plantilla
  `.env.example` del checkout de infraestructura sin versionar valores reales.
- **Trazas por validar**: activar `observability` y el agente OTel no constituye evidencia de ingesta. Recoger
  un `trace_id` real y sus spans HTTP/JDBC/AMQP en Tempo, además del scrape Prometheus y la ingesta Loki.
  `X-Correlation-Id` no es un `trace_id`: su eco HTTP puede comprobarse sin que exista una traza distribuida.

### Comprobación y resultados de la entrega IAM

El changelog de entrada DB es `changelog/changelog-master.yaml`. El Service conserva `ddl-auto: validate` y no crea tablas al arrancar. La base oficial ya incluye outbox y columnas de contraseña temporal; la PR DB #79 añade preflight/CI/documentación, sin DDL nuevo.

Con las variables PostgreSQL habituales configuradas, ejecutar desde el checkout DB:

```bash
psql --no-psqlrc -v ON_ERROR_STOP=1 -f scripts/verify-iam-integration.sql
```

El preflight es de solo lectura y verifica esquema, índices, seeds y grupos `iam_reader`/`iam_writer`/`iam_admin` de tipo `NOLOGIN`. No crea ni autentica el usuario de aplicación; provisionar su pertenencia y verificar permisos efectivos por separado.

El ensayo registrado usó la base aislada `iam_delivery_20261021`, contenedor `iam-r3-demo-20261020`, puerto PostgreSQL `55432` y su propietario, sin modificar la base compartida. Se aplicaron 37 changesets heredados; validate aprobó y el segundo update ejecutó 0 nuevos. El preflight aprobó el esquema completo y rechazó la demo anterior por ausencia de `identity.outbox`. Ese `ROLLBACK` final del script de lectura no acredita un rollback de migraciones ni atomicidad ante fallo de negocio.

El JAR local en `127.0.0.1:18080` respondió 200 en health/readiness/Prometheus; 54 IT PostgreSQL aprobaron y uno de broker se omitió. Ese puerto es un acceso directo temporal del ensayo, distinto del edge de Compose. El intento de imagen Docker falló al resolver `production.cloudfront.docker.com` al descargar la base; no quedó una imagen nueva validada.

Para MFE, la URL base `VITE_IAM_API_URL` se configura según la ruta publicada por el entorno; el adaptador añade `/auth/login` y `/auth/me`. En desarrollo, `IAM_API_PROXY_TARGET` determina el destino del proxy Vite. El smoke registrado usó Vite `15101` → JAR `18080` → DB local; 46 pruebas MFE y el build federado aprobaron. No montó el shell en navegador ni probó consumidores RabbitMQ o ingesta real de telemetría. Las instrucciones/versiones completas están en la [guía MFE](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/docs/integration/iam-service.md).

La CI MFE requiere el secreto `FRONTEND_DEPENDENCIES_READ_TOKEN` para leer frontend-platform/design-system privados; no es configuración del bundle ni debe exponerse como variable `VITE_*`. Su ausencia detuvo CI antes de instalación y pruebas.

### Levantar notification-service local (perfil `notification`)

Simétrico a `iam`. El perfil `notification` levanta `notification-api` + `notification-worker` (Go) + `rabbitmq` + `mailhog` + `traefik` (edge).

```bash
# 1. Levanta postgres + notification-api + notification-worker + rabbitmq + mailhog + traefik.
#    --build para no correr imágenes stale (mismo gotcha que iam).
docker compose --env-file .env.develop --profile notification up -d --build

# 2. Aplicar el schema si aplica (perfil tooling)
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> update

# 3. Con trazas E2E (Tempo/Grafana, ADR-008):
docker compose --env-file .env.develop --profile notification --profile observability up -d --build
```

- **RabbitMQ — reset dirigido del volumen:** el broker se aprovisiona desde `definitions.json`, que **debe incluir `users` y `permissions`**; si falta un usuario/permiso o cambió el aprovisionamiento, **reseteá el volumen de RabbitMQ** (no el de Postgres) para que reaplique las definitions:

  ```bash
  docker compose --env-file .env.develop rm -sfv rabbitmq   # borra solo el contenedor+volumen de rabbitmq
  docker compose --env-file .env.develop --profile notification up -d --build
  ```

  > **No uses `down -v`** para esto: borraría también el volumen de Postgres y perderías el schema/seed. Reseteá **solo** `rabbitmq`.
- **MailHog:** captura el correo saliente del worker para inspección local (no envía a destinatarios reales).
- **Acceso:** igual que iam, solo por el edge Traefik; los `-api` no se exponen al host.

---

## Gotchas de observabilidad / deploy

- **OTel — el puerto depende del stack, no lo copies entre servicios.** El endpoint OTLP correcto **difiere por runtime**:
  - **`iam` (agent Java):** `:4318` con **http/protobuf**.
  - **`notification` (SDK Go):** `:4317` con **gRPC**.

  **No copies la config OTLP de `notification` a un micro Java.** Apuntar el `iam` (Java) a `:4317` produjo `Connection reset` → **0 trazas** en Tempo, sin error visible en el arranque. Si un servicio no emite trazas, revisá primero puerto + protocolo contra su runtime.
- **Bug del socket de Docker Desktop / WSL rompe el edge Traefik en local.** El bind-mount del socket de Docker en Docker Desktop sobre WSL no siempre expone un socket real, y el provider Docker de Traefik queda sin descubrir servicios → el edge no enruta. **Workaround:** usar el **provider `file`** de Traefik (configuración estática de rutas) en lugar del provider Docker, o correr en un entorno con socket de Docker real. **Estado: `E2-edge` no validado localmente** por este bug — el enrutado por edge se ha verificado por otras vías, no por el descubrimiento automático vía socket.

---

### Puertos expuestos al host por ambiente (ADR-007)

| Recurso | develop | qa | staging | main |
|---|---|---|---|---|
| Edge (Traefik) — único entry point | 18080 | 18081 | 18082 | 18083 |
| Traefik dashboard | 18090 | 18091 | 18092 | 18093 |
| Postgres | 15432 | 15433 | 15434 | 15435 |
| RabbitMQ mgmt | 15672 | 15673 | 15674 | 15675 |
| Grafana / Tempo / Prometheus / Loki (ADR-008) | base + offset | +1 | +2 | +3 |
| `iam-api` (y cualquier `-api` de dominio) | **no expuesto** — solo vía edge | | | |
