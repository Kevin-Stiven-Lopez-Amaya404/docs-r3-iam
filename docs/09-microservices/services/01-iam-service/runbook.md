# Runbook — iam-service

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Puerto de aplicación: `8080` interno; acceso previsto por edge `/iam/*` (ADR-007). El smoke R3 usó un JAR aislado en loopback `18080`. | SLO objetivo: 99.9 % uptime | Criticidad: P0

> Esta versión reemplaza el borrador previo (2026-06-20), que documentaba un diseño no
> implementado (`JWT_SECRET` HMAC + Redis para revocación). La implementación real
> (HU-IAM-001..011) usa **RS256 + JWKS** y **no tiene Redis**; ver
> [decisions.md](./decisions.md) DEC-001/DEC-008 y [authentication.md](../../../07-api/authentication.md).

---

## Versiones y alcance de operación

La ampliación R3 corresponde a Service `7c837e0824263cef0aef89f807731adca839f25e`,
[PR #43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta
hacia `develop`. DB [#79](https://github.com/code-sena/design-software-iam-db/pull/79)
y MFE [#10](https://github.com/code-sena/design-software-iam-mfe/pull/10) también
estaban abiertas al corte. Los comandos de este runbook son procedimientos;
su inclusión no acredita que se hayan ejecutado en el entorno compartido.
El [informe del plan](../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md)
conserva resultados históricos y límites, incluida la CI MFE sin credencial de
lectura de dependencias privadas.

## Healthcheck

> Actualizado en HU-IAM-011 (implementación) y HU-IAM-012 (acceso vía edge). El servicio expone
> un `HealthController` propio (`co.sena.iam.adapter.in.rest`) — **no** Spring Boot Actuator: la
> rama R3 tiene `management.endpoints.web.exposure.include: health,info,prometheus`, pero
> `/health` y `/ready` a continuación son el contrato real que opera el healthcheck de
> orquestación, no `/actuator/health`.

| Endpoint (interno, en el contenedor) | Vía edge (`docker-infra`, perfil `iam` + Traefik, ADR-007) | Respuesta esperada | Cuándo falla |
|---|---|---|---|
| `GET /health` | `GET http://localhost:${EDGE_PORT}/iam/health` | `200 { "status": "UP" }` — liveness (el proceso vive) | Proceso caído o sin red |
| `GET /ready` | `GET http://localhost:${EDGE_PORT}/iam/ready` | `200 { "status": "READY", "db": "ok"[, "broker": "ok"] }` | BD no conecta, o el broker configurado no conecta → `503 { "status": "NOT_READY", "db": "ok"\|"down"[, "broker": "down"] }` |

**SLO**: `GET /health` debe responder en < 200 ms (p99). `GET /ready` en < 500 ms (p99).

`/health` responde `200` aunque la BD o el broker estén caídos — solo indica que el proceso vive (no depende de
ninguna dependencia externa). `/ready` ejecuta `SELECT 1` contra Postgres (siempre — sin BD el servicio no
puede autenticar) y, **solo si `spring.rabbitmq.host` está configurado**, comprueba también el broker del
outbox relay de HU-IAM-007 (`BrokerHealthConfiguration` → `BrokerReadinessProbe`, gateado igual que el relay;
si está intencionalmente apagado, `/ready` no depende de un broker inexistente y el campo `broker` no aparece
en la respuesta). `503` si algún chequeo incluido falla.

`/health` y `/ready` son endpoints operacionales en la raíz de la app y sí
figuran en [OpenAPI IAM](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/contracts/openapi/iam.yaml). En el
perfil edge previsto, el router `/iam` con `StripPrefix` transforma
`/iam/api/v1/auth/login` en `/api/v1/auth/login` y `/iam/health` en `/health`.
Esta entrega verificó las sondas directamente contra el JAR local; no acredita
ese recorrido edge ni la pila completa. `/actuator/prometheus` debe permanecer
accesible por la red interna de observabilidad y excluirse de la ruta pública.

Un broker caído pone `/ready` en `503` (con `"broker": "down"`) pero **no bloquea el login**: el outbox relay
(HU-IAM-007) solo publica a `iam-events` cuando el broker está disponible; si no lo está, los eventos se
quedan **acumulados sin pérdida** en `identity.outbox` hasta que el broker vuelve (patrón Outbox, ver
[ADR-001](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-001-message-broker.md)).

---

## Observabilidad — base HU-IAM-011 y ampliación R3

- **Traces**: OTel Java agent (auto-instrumentación, zero-code) — spans HTTP (Spring MVC) + spans de BD (JDBC/Hibernate), correlacionados por `trace_id`. Se activa vía `-javaagent` en el `Dockerfile` (imagen ya lo descarga en build); se configura 100 % por variables de entorno, sin secretos:

  | Variable | Ejemplo | Descripción |
  |----------|---------|-------------|
  | `OTEL_EXPORTER_OTLP_ENDPOINT` | `http://otel-collector:4318` | Endpoint OTLP del Collector del ambiente. |
  | `OTEL_EXPORTER_OTLP_PROTOCOL` | `http/protobuf` | El agent Java exporta OTLP por `http/protobuf` (puerto `4318`), no gRPC (`4317`) — usar gRPC en `4317` da `Connection reset`. Verificado en docker-infra (HU-IAM-012 fix). |
  | `OTEL_SERVICE_NAME` | `iam-service` | Nombre recomendado en la guía R3. La configuración histórica de infraestructura usa `iam-api`; acordar el nombre del entorno antes de consultar trazas. La etiqueta Micrometer `service="iam-service"` procede de `spring.application.name`, independientemente de este valor. |
  | `OTEL_RESOURCE_ATTRIBUTES` | `deployment.environment=develop` | Alinea `deployment.environment` por ambiente (ADR-008 §6). |

  Configurar un colector alcanzable y cargar el agente para exportar al entorno.
  La disponibilidad de la API no acredita exportación ni recepción OTLP; el
  smoke R3 del JAR se ejecutó sin agente y no demuestra trazas HTTP/JDBC.

- **Logs**: la configuración JSON por stdout y el agente OTel son previos a R3.
  Con span activo, el agente aporta `trace_id`/`span_id` al MDC. El filtro nuevo
  valida `X-Correlation-Id` con `[A-Za-z0-9._-]{1,128}`, genera UUID si falta o
  no es válido, devuelve el ID y restaura el MDC al terminar, incluso ante error.
  `ApiErrors` prioriza `trace_id` activo, luego `X-Request-Id` y UUID. Esta
  correlación de petición no sustituye W3C `traceparent` ni prueba ingesta Loki.

- **Métricas**: no formaban parte de la HU-IAM-011 histórica. La ampliación R3
  añade Micrometer Prometheus y expone contador e histograma HTTP, con etiqueta
  `service="iam-service"`, mediante `GET /actuator/prometheus`. No acredita un
  scrape externo, dashboard o disparo/recuperación de alertas.

### Comprobación local y resultados conservados

Con el JAR configurado y arrancado localmente, adaptar el puerto si corresponde:

```bash
curl -i -H 'X-Correlation-Id: entrega-iam-20261021' http://127.0.0.1:18080/health
curl -i http://127.0.0.1:18080/ready
curl -s http://127.0.0.1:18080/actuator/prometheus
```

El registro local del 20 de octubre conserva HTTP 200 en esas tres rutas, el ID
enviado y las series `http_server_requests_seconds_count` y
`http_server_requests_seconds_bucket` con `service="iam-service"`. La suite
local aprobó 186 casos de servicio y 54 IT PostgreSQL; una IT RabbitMQ se omitió
por falta de `IAM_BROKER_HOST`. La CI de Service aprobó los 186 y omitió las 55
IT por ausencia de dependencias externas. El build Docker falló antes de compilar
por DNS al descargar `eclipse-temurin:21-jre`; no hay imagen nueva validada.
Consultar [validación de la implementación](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/docs/observability/validation.md)
y la [matriz central](../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md).

### Compatibilidad de persistencia antes del arranque

Provisionar primero el changelog oficial `changelog/changelog-master.yaml`.
La validación local utilizó DB `a1e01a42692da77269568f0ef49b2e4d3d035fa3`, 37
changesets, en una base aislada. El servicio conserva `ddl-auto: validate` y
requiere `identity.outbox` y las columnas temporales ya existentes del modelo.
El [preflight de DB](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/scripts/verify-iam-integration.sql)
comprueba compatibilidad en solo lectura; no crea tablas ni demuestra por sí
solo acceso efectivo del usuario de aplicación. Las pruebas usaron el propietario
de la base aislada, no `design_software_app`; ese acceso real sigue pendiente.

---

## Variables de entorno requeridas

Sin defaults de secreto: si falta una obligatoria, el servicio **debe fallar al arrancar**, nunca arrancar con
un valor adivinable.

| Variable | Obligatoria | Ejemplo | Descripción |
|----------|:---:|---------|-------------|
| `IAM_DB_DSN` | Sí | `jdbc:postgresql://postgres:5432/design-software-develop` | JDBC URL a la BD compartida (schemas `identity`/`identity_audit`/`session`/`rbac`/`rbac_catalog`). |
| `IAM_DB_USERNAME` | Sí | `design_software_app` | Usuario de aplicación least-privilege (`iam_writer`, ver `docker-infra/create-app-user.sh`). |
| `IAM_DB_PASSWORD` | Sí | — | Password del usuario de aplicación. |
| `IAM_JWT_PRIVATE_KEY` | Sí | PEM o base64 PKCS#8 de una clave RSA | Clave privada RS256 para firmar access tokens. El servicio **rehúsa arrancar** sin ella. La pública se deriva y se publica en JWKS — nunca se configura por separado. |
| `IAM_JWT_ISSUER` | Sí | `https://iam.design-software.local` | Claim `iss` de los tokens emitidos; los verificadores downstream lo validan. |
| `IAM_JWT_KEY_ID` | No (default `iam-rsa-1`) | `iam-rsa-2026-08` | `kid` publicado en JWKS para esta clave. |
| `IAM_JWT_PREVIOUS_PUBLIC_KEY` / `IAM_JWT_PREVIOUS_KEY_ID` | No | — | Solo durante una ventana de rotación (DEC-008): clave pública saliente + su `kid`, para que JWKS siga publicando ambas hasta que expiren los tokens firmados con la anterior (≤ 15 min). |
| `SPRING_RABBITMQ_HOST` | No* | `rabbitmq` | Host del broker. **Si no está seteada, el outbox relay queda apagado** (no publica; los eventos se acumulan en `identity.outbox`) — no es un fallo de arranque. |
| `SPRING_RABBITMQ_USERNAME` / `SPRING_RABBITMQ_PASSWORD` | Solo si `SPRING_RABBITMQ_HOST` está seteada | `app` / `app` | Credenciales del broker (perfil `broker`/`iam` de docker-infra). |
| `SPRING_RABBITMQ_PORT` / `SPRING_RABBITMQ_VIRTUAL_HOST` | No | `5672` / `/` | Solo si se desvían de los defaults de Spring AMQP. |
| `IAM_EVENTS_EXCHANGE` | No (default `iam-events`) | `iam-events` | Exchange topic (declarado como bean idempotente al arrancar — no depende de `definitions.json` del broker). |
| `IAM_OUTBOX_RELAY_POLL_INTERVAL_MS` / `_INITIAL_DELAY_MS` / `_BATCH_SIZE` | No | `5000` / `5000` / `100` | Tuning del poller del relay. |
| `OTEL_EXPORTER_OTLP_ENDPOINT` / `OTEL_EXPORTER_OTLP_PROTOCOL` | No para el arranque; requeridas para el destino de observabilidad acordado | `http://otel-collector:4318` / `http/protobuf` | Configuración del agente existente (ADR-008); comprobar recepción en el colector. |
| `OTEL_SERVICE_NAME` | No (default lo toma del agente) | `iam-service` | Nombre OTel recomendado en R3; comprobar el valor del entorno. No cambia la etiqueta Micrometer `service`. |

> \* Al menos en `develop`/`qa`/`staging`/`main` se espera el broker configurado (perfil `iam` de docker-infra
> lo trae por defecto) para que HU-IAM-007 (eventos → `iam-events`) funcione E2E; queda opcional a nivel de
> arranque solo para no acoplar los tests de slice/IT que no requieren broker.

**Cómo levantarlo en local (docker-infra, perfil `iam`)**: ver
[10-devops/local-setup.md](../../../10-devops/local-setup.md) §Perfil `iam`.

---

## Alertas críticas

Umbrales y acciones de operación previstos. Esta entrega no conserva evidencia
de disparo/recuperación de una alerta IAM ni valida los contactos de guardia.

| Alerta | Condición de disparo | Severidad | Impacto si no se atiende | Acción inmediata |
|--------|---------------------|-----------|--------------------------|------------------|
| **BD no responde** | Timeout al conectar, o `/ready` devuelve `503` con `"db": "down"` | P0 | Ningún usuario puede autenticarse ni el RBAC resuelve. Todo el sistema queda bloqueado. | 1. Verificar el contenedor/pod de PostgreSQL. 2. Revisar logs de BD. 3. Si la BD está sana, revisar `IAM_DB_DSN`/`IAM_DB_USERNAME`/red interna. 4. Si persiste > 5 min, escalar a infra. |
| **Broker no responde** | `/ready` devuelve `503` con `"broker": "down"` (solo si `SPRING_RABBITMQ_HOST` está configurada), o el outbox relay deja de drenar (`identity.outbox` con filas sin `published_at` creciendo) | P1 | Login/RBAC siguen funcionando con normalidad. Los eventos (`iam.user.created`, `iam.session.started`, etc.) **no se pierden** (quedan en `identity.outbox`) pero no llegan a consumidores hasta que el broker vuelve. | 1. Verificar RabbitMQ (`docker compose ps rabbitmq`). 2. Revisar `SPRING_RABBITMQ_*`. 3. Al reconectar, el relay drena el backlog automáticamente (no requiere reinicio de iam-api). 4. Si el backlog no drena en 15 min con broker sano, revisar logs de `OutboxRelay`/`RabbitEventPublisher`. |
| **Error rate > 5 % en 5 min** | Más del 5 % de requests a `/api/v1/auth/*` retornan 5xx en ventana deslizante de 5 min | P1 | Degradación del flujo de autenticación para usuarios activos. | 1. Revisar logs (JSON estructurado con `trace_id`, ADR-008) — correlacionar en Grafana/Tempo. 2. Verificar conectividad a BD/broker. 3. Si es pool agotado, escalar réplicas o reiniciar. |
| **`IAM_JWT_PRIVATE_KEY` comprometida** | Sospecha o confirmación de compromiso de la clave privada activa | P0 | Un atacante puede firmar tokens RS256 válidos con cualquier claim arbitrario. | Ver **Rotación de la clave de firma (DEC-008)** más abajo. No basta con reiniciar con una clave nueva sin coordinar la ventana de JWKS — servicios downstream cachean la clave pública. |
| **Account lockout masivo** | > 20 cuentas bloqueadas (`locked_until IS NOT NULL`) en 10 min | P2 | Posible ataque de fuerza bruta/credential stuffing. Usuarios legítimos pueden quedar bloqueados. | 1. Revisar `identity_audit.audit_login` por IP. 2. Evaluar bloqueo de IP a nivel de red. 3. Notificar a seguridad. 4. Desbloqueo manual solo para roles críticos tras validar identidad. |

---

## Procedimientos comunes

### Reinicio del servicio

```bash
# Docker Compose (perfil iam, docker-infra)
docker compose --profile iam restart iam-api

# Verificar que levantó correctamente
docker compose --profile iam logs --tail=50 iam-api

# Kubernetes — rollout restart (sin downtime si hay >= 2 réplicas)
kubectl rollout restart deployment/iam-api -n <namespace>
kubectl rollout status deployment/iam-api -n <namespace>
```

> Un reinicio **no invalida** los access tokens vigentes (expiran solos, 15 min) ni los refresh tokens en BD.
> No hay lista de revocación en memoria/Redis que perder — DEC-001 acepta esto como trade-off conocido.

---

### Revisión de logs de autenticación fallida

```bash
# Docker Compose — últimas 200 líneas filtrando outcomes de auth
docker compose --profile iam logs --tail=200 iam-api | grep -E "INVALID_PASSWORD|USER_NOT_FOUND|ACCOUNT_LOCKED|TOKEN_EXPIRED"

# Intentos fallidos por IP (últimos 30 min) directamente en BD
psql "$IAM_DB_DSN" -c "
  SELECT ip_address, COUNT(*) AS intentos, COUNT(DISTINCT email_attempted) AS emails_distintos,
         MAX(attempted_at) AS ultimo_intento
  FROM identity_audit.audit_login
  WHERE outcome != 'SUCCESS' AND attempted_at > NOW() - INTERVAL '30 minutes'
  GROUP BY ip_address ORDER BY intentos DESC LIMIT 20;
"
```

---

### Desbloquear cuenta de usuario manualmente

Usar solo cuando el bloqueo es por falsos positivos confirmados, o tras validar la identidad por canal alterno.

```sql
-- 1. Verificar estado actual
SELECT id, email, failed_attempts, locked_until, is_active
FROM identity."user" WHERE email = 'usuario@ejemplo.com';

-- 2. Desbloquear
UPDATE identity."user"
SET failed_attempts = 0, locked_until = NULL, updated_at = NOW()
WHERE email = 'usuario@ejemplo.com' AND is_active = true;
```

> Si `is_active = false` la cuenta está desactivada (no es un bloqueo por intentos); reactivarla requiere
> `SYSTEM_ADMIN` o `CENTER_DIRECTOR` con scope sobre el centro del usuario (HU-IAM-005).

---

### Revocar todas las sesiones de un usuario

```sql
UPDATE session.refresh_token
SET is_revoked = true, revoked_at = NOW()
WHERE user_id = (SELECT id FROM identity."user" WHERE email = 'usuario@ejemplo.com')
  AND is_revoked = false AND expires_at > NOW();
```

> El access token vigente del usuario **sigue siendo válido hasta que expira** (máx. 15 min) — no hay
> revocación de access tokens en curso (DEC-001). Esto es el diseño aceptado, no un bug operativo.

---

## Escenarios de falla y recuperación

### BD no conecta

**Síntomas**: `/ready` → `503` con `"db": "down"`. Todo `/api/v1/auth/*` retorna 5xx.

**Pasos**:
1. Verificar el contenedor/pod de PostgreSQL (`docker compose ps postgres` / `kubectl get pods -l app=postgres`).
2. Probar conectividad desde el pod de IAM.
3. Si la BD está sana, revisar `IAM_DB_DSN`/`IAM_DB_USERNAME`/`IAM_DB_PASSWORD` y red interna.
4. Si la BD estaba caída y vuelve: el pool de HikariCP se recupera solo. Si no reconecta en 60 s, reiniciar `iam-api`.

**RTO objetivo**: < 15 min si la causa es reinicio de BD.

---

### Broker no conecta (RabbitMQ)

**Síntomas**: `/ready` → `503` con `"broker": "down"` (solo si `SPRING_RABBITMQ_HOST` está configurada), pero
login/RBAC siguen funcionando. Filas sin `published_at` creciendo en `identity.outbox`.

**Comportamiento en modo degradado**: login, refresh, RBAC y todo lo síncrono funcionan sin cambios. Solo se
detiene la **publicación** de eventos (`iam.user.created`, `iam.user.deactivated`, `iam.role.assigned`,
`iam.session.started`) — quedan encolados en `identity.outbox` sin pérdida.

**Pasos**:
1. Verificar RabbitMQ (`docker compose --profile iam ps rabbitmq`; `rabbitmq-diagnostics ping`).
2. Si RabbitMQ está caído sin causa obvia, reiniciarlo. **Reset quirúrgico si hace falta recrear el volumen**:
   `docker compose --profile iam stop rabbitmq && docker volume rm <proyecto>_rabbitmq_data && docker compose --profile iam up rabbitmq` —
   **nunca** `docker compose down -v` (borraría también el volumen de Postgres compartido con todos los módulos).
3. Al volver, el `OutboxRelay` drena el backlog en el siguiente ciclo de poll (`IAM_OUTBOX_RELAY_POLL_INTERVAL_MS`, default 5 s) — no requiere reiniciar `iam-api`.
4. Confirmar drenado: `SELECT count(*) FROM identity.outbox WHERE published_at IS NULL;` debe volver a 0.

---

### Rotación de la clave de firma (DEC-008 — JWKS, sin downtime)

> A diferencia de un secreto HMAC compartido, RS256 + JWKS permite rotar **sin invalidar sesiones activas**:
> se publican ambas claves (`kid` actual + `kid` saliente) durante la ventana de transición.

```bash
# 1. Generar el nuevo par RSA (PKCS#8)
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out new-private.pem
openssl pkey -in new-private.pem -pubout -out new-public.pem

# 2. Desplegar con AMBAS claves configuradas: la nueva como activa, la saliente como "previous"
#    IAM_JWT_PRIVATE_KEY        = contenido de new-private.pem
#    IAM_JWT_KEY_ID             = nuevo kid, p. ej. iam-rsa-2026-09
#    IAM_JWT_PREVIOUS_PUBLIC_KEY= clave pública de la clave SALIENTE (la que era IAM_JWT_PRIVATE_KEY antes)
#    IAM_JWT_PREVIOUS_KEY_ID    = kid de la clave saliente
kubectl rollout restart deployment/iam-api -n <namespace>

# 3. Verificar que JWKS publica ambas entradas
curl -s http://localhost:${EDGE_PORT}/iam/api/v1/auth/.well-known/jwks.json | jq '.keys[].kid'

# 4. Esperar la ventana de transición (>= vida del access token, 15 min) para que
#    expiren naturalmente los tokens firmados con la clave saliente.

# 5. Retirar la clave saliente: quitar IAM_JWT_PREVIOUS_PUBLIC_KEY / IAM_JWT_PREVIOUS_KEY_ID y reiniciar.
```

**Registrar el incidente/rotación**: fecha, motivo (programada trimestral vs. compromiso), quién ejecutó.

---

## Escalamiento

| Condición | Paso siguiente | Contacto |
|-----------|---------------|----------|
| BD no vuelve en 15 min después de reinicio | Escalar a infra con logs de PostgreSQL adjuntos | @infra-oncall |
| Broker no vuelve en 30 min y `identity.outbox` sigue creciendo | Escalar a infra; evaluar impacto en consumidores de `iam-events` | @infra-oncall |
| Error rate > 5 % sostenido > 10 min sin causa clara | Escalar a tech lead con trazas (Tempo) y logs de los últimos 15 min | @tech-lead |
| Clave de firma comprometida (sospecha o confirmado) | Activar protocolo de incidente de seguridad P0 **antes** de cualquier acción técnica | @security-lead, @tech-lead |
| Account lockout masivo (> 20 cuentas en 10 min) | Notificar a seguridad; no desbloquear hasta confirmar que no es ataque activo | @security-lead |
| Servicio no levanta después de 2 reinicios | Revisar env vars/secretos/logs de arranque; escalar a tech lead si no se identifica causa | @tech-lead |
