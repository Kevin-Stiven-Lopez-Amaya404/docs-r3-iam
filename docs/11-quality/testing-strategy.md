# Estrategia de pruebas

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Calidad

Estrategia de pruebas para los microservicios del sistema **Horarios SENA**, de stack políglota Java + Go ([ADR-006](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-006-polyglot-stack-strategy.md)). Los estados de construcción de agosto se conservan como antecedentes; esta actualización verifica únicamente la entrega IAM Service/DB/MFE del 21 de octubre y no certifica el estado actual de los demás servicios.

- **Entrega IAM en ramas/PR abiertas:**186 pruebas backend aprobadas; PostgreSQL 54 aprobadas/1 omitida; MFE 46 aprobadas y build local. DB 37 changesets heredados, preflight e incremental sin cambios comprobados. Detalle por versión/caso en el [expediente IAM](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).
- **Antecedentes de otros servicios:** mantienen sus pruebas documentadas en sus expedientes; no se infiere ausencia o existencia actual de aplicación de servicios no revisados.

La pirámide describe los controles esperados; la tabla distingue evidencia actual IAM de pruebas históricas/objetivos. Una prueba omitida no cuenta como aprobada y el agregado de una suite no acredita cada criterio de aceptación.

## Principios

- **Prueba lo que existe.** Para los servicios que solo tienen capa de datos, el foco es migraciones, DDL, seeds y restauración. Para los que ya tienen capa de aplicación (`iam`, `notification`), aplican los cuatro niveles superiores de la pirámide con evidencia real.
- **Aislamiento por servicio.** Cada `*-db` se prueba de forma independiente, con su propio schema y su propio tracking de Liquibase (ver [migration-strategy.md](../06-data/migration-strategy.md)).
- **Idempotencia y reversibilidad.** Toda migración debe poder aplicarse, revertirse y re-aplicarse en un entorno limpio sin intervención manual.
- **Rápido y determinista.** Las pruebas corren sobre contenedores efímeros de PostgreSQL 16 y no dependen de datos preexistentes ni de orden entre servicios.

## Pirámide de pruebas

De más numerosas/baratas (base) a más costosas (cima). Los estados se refieren a la evidencia revisada IAM; el baseline de otros componentes no se reejecutó.

| Nivel | Qué valida | Estado hoy |
|-------|------------|------------|
| Migraciones y DDL | Creación, validación y estrategia de reversibilidad | IAM:37 cambios desde cero y segundo update sin cambios, local/CI. Rollback no ejecutado en este plan. |
| Datos semilla (seeds) | Catálogos requeridos e idempotencia | Preflight IAM valida seeds; repetir update no equivale a ejecutar cada INSERT dos veces. |
| Unitarias/contexto/UI | Lógica de negocio y adaptadores | Service 186 y MFE 46 aprobadas; son suites totales, no cantidades de tests nuevos. |
| Contrato entre servicios | Compatibilidad productor/consumidor | Consumidor MFE alineado a OpenAPI; gate Pact externo no acreditado en esta entrega. |
| Integración | Servicio con PostgreSQL y dependencias | Local IAM:54 aprobadas,1 omitida por broker. CI Service omite55IT. |
| End-to-end | Navegador/shell/edge/servicios/broker/consumidor | Smoke HTTP adaptador/proxy/API/DB local; recorrido completo no acreditado. |

## Nivel 1 — Pruebas de migraciones Liquibase (hoy)

Cada repo `*-db` usa su changelog y el perfil `tooling` de Docker Compose (ver [local-setup.md](../10-devops/local-setup.md)). Para IAM, la validación de este plan usa `changelog/changelog-master.yaml` en una base aislada; los casos siguientes son controles de estrategia y no deben presentarse todos como ejecutados aquí.

Casos mínimos por servicio:

1. **Forward limpio.** Sobre una BD vacía, `update` aplica todo el changelog sin error y respetando el orden `00_extensions → 01_schemas → 02_types → 03_tables → 04_alter (FKs) → 05_views…08_triggers → 10_indexes → 02_dml (seeds) → 03_dcl → 04_tcl`.
2. **Rollback.** Cada changeset declara su `rollback` espejo en `05_rollbacks/`; se valida `rollback <tag>` hasta dejar la BD en el estado anterior. Los rollbacks se prueban **en local**; en `qa`/`staging`/`main` las migraciones son forward-only.
3. **Re-aplicación (idempotencia de esquema).** Tras un rollback, un nuevo `update` vuelve a dejar el esquema íntegro.
4. **`runOnChange` en lógica almacenada.** Vistas, funciones, procedimientos y triggers (`05_views`/`06_functions`/`07_procedures`/`08_triggers`) marcados con `runOnChange: true` se re-aplican al cambiar su definición.
5. **Integridad referencial.** Las FKs creadas en `04_alter` existen con la acción `ON UPDATE`/`ON DELETE` correcta y hay un índice por cada FK.

Comandos de referencia (un módulo a la vez; módulos: `liquibase-academic`, `liquibase-actors`, `liquibase-audit`, `liquibase-document`, `liquibase-iam`, `liquibase-monitoring`, `liquibase-reference`, `liquibase-scheduling`, `liquibase-training`):

```bash
# Levantar Postgres del ambiente de pruebas
docker compose --env-file .env.develop up postgres -d

# Aplicar y verificar el estado de un módulo
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> update
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> status --verbose

# Validar reversibilidad en local
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> rollback <tag>
```

> Buena práctica: usar `liquibase validate` para detectar changesets con `id`/`author` duplicados o `rollback` faltante antes de abrir el PR.

## Nivel 2 — Datos de prueba y seeds (hoy)

- Los **seeds de catálogo** viven en `02_dml/` y son **idempotentes** (`INSERT ... ON CONFLICT DO NOTHING` o `MERGE`), de modo que re-ejecutarlos no falla ni duplica. La prueba consiste en aplicar el seed dos veces y verificar conteos estables.
- Los **datos de prueba** (fixtures que no deben llegar a `staging`/`main`) se aíslan con `context`/`labels` de Liquibase, de modo que un `update` en ambientes compartidos no los cargue.
- Para escenarios que necesiten datos ficticios, usar valores seguros (ver [security-rules.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/security-rules.md)); nunca datos personales reales de aprendices o instructores.

## Nivel 3 — Unitarias de la capa de aplicación (ACTIVO en `iam` y `notification`)

Cubren lógica de negocio pura con dependencias externas simuladas. Ya activo:

- **`iam-service` (Java/Spring):** suite revisada de 186 pruebas (core 92/adapters 90/API 4), con12 casos nuevos de observabilidad y regresiones existentes de autenticación/RBAC. Los54IT PostgreSQL aprobados se registran por separado.
- **`notification-service` (Go):** unitarias sobre resolución de destinatario, render de plantilla y máquina de reintentos.
- Para servicios que todavía no tengan capa de aplicación, al construirla: cubrir validaciones, máquinas de estado parametrizables de ADR-004 y cálculo de KPIs cuando corresponda. `scheduling-engine-workflow` y `conflict-validator-worker` (validación de choques de horario) son candidatos por su densidad de reglas; esta actualización no vuelve a auditar su estado.
- Umbral de cobertura objetivo: **punto abierto**, a fijar en el gate de [ci-cd.md](../10-devops/ci-cd.md).

> ⚠️ **Las unitarias no bastan por sí solas.** Dos incidentes reales confirmaron que suites unitarias verdes dejaron pasar defectos de arranque y de integración; ver las reglas de **contexto completo (Java)** y **E2E con broker/BD fresco (Go)** más abajo.

### Regla de contexto completo (Java) — `@SpringBootTest` obligatorio

Todo servicio Java debe tener **al menos un `@SpringBootTest` de contexto completo** (*context-loads*) que arranque el `web` environment real, **además** de los `@WebMvcTest`/unitarios por controlador.

- **Incidente que lo motiva (ambiguous-mapping):** dos `@RestController` implementaban la misma interfaz OpenAPI generada (`UsersApi`) → Spring falló al arrancar por *ambiguous mapping*. Los `@WebMvcTest` y las unitarias **no lo cazaron**, porque cargan un solo controlador aislado y nunca levantan el contexto web completo.
- **Remediación adoptada:** se agregó `IamApiApplicationContextTest`, un test *full-boot* que arranca el contexto entero; solo esa clase detecta el choque de mapeos.
- **Regla derivada:** una interfaz OpenAPI generada = **un único** `@RestController` que la implementa, y **un** test de contexto completo por servicio Java que garantice que el ApplicationContext levanta.

### Regla Go / eventos — E2E con broker y BD frescos

En los servicios Go orientados a eventos, las **unitarias mockean el use case**, así que no ejercitan el camino real de consumo/entrega. La validación de aceptación exige un **E2E con broker (RabbitMQ) y BD frescos**.

- **Incidente que lo motiva (notification HU-004):** el use case devolvía `nil`; los unit tests (que mockean el use case) pasaban en verde. **Solo el E2E** con broker/BD limpios detectó el defecto.
- **Regla derivada:** ninguna HU de consumo/entrega de eventos se da por *done* con solo unitarias; requiere E2E real que cubra happy path, DLQ e idempotencia por `event_id`.

## Nivel 4 — Pruebas de contrato entre servicios (ACTIVO — gate Pact)

La arquitectura es orientada a eventos (broker, ADR-001) con read models (ADR-002). Pact se documentó como gate de compatibilidad productor/consumidor en el baseline anterior. Este plan no conserva una ejecución Pact; no se lo declara aprobado como parte de sus CI.

- **Contrato existente:** eventos IAM en `iam-events` y consumo AMQP de notification. La regresión local comprobó eventos staged en outbox; con broker ausente no acredita publicación/consumo ni sustituye Pact.
- Validar el **envelope de eventos** y el `event_type` (ej. `scheduling.class_session.created`) contra el [event-catalog.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/event-catalog.md).
- Verificar **idempotencia de consumo**: `notification`, `audit` y otros consumidores deben tolerar re-entrega at-least-once (unicidad por `event_id` / `source_event_id`).
- Enfoque consumer-driven contract entre cada par publicador/consumidor, para desacoplar despliegues.

> **Deuda de contrato conocida:** en `shared-contracts` conviven un *envelope plano* y uno *anidado* (`error-codes.yaml` vs `_shared.yaml`); además el Outbox persiste la carga bajo `data` y el relay la renombra a `payload` al publicar. Las pruebas de contrato deben fijar cuál es la forma canónica; ver [shared-contracts](../07-api/) y la skill de contratos.

## Nivel 5 — Integración y E2E (ACTIVO en `iam` y `notification`)

- **Integración IAM revisada:**55 casos Failsafe,54 aprobados y1 omitido, agrupados en nueve XML. Incluye login/audit/refresh/estado, recuperación, RBAC y administración. Usuario propietario de base aislada, no usuario real del ambiente compartido.
- **Smoke HTTP MFE revisado:** adaptador real por Vite/proxy → JAR → DB; sesión válida, correlación/identidad correctas y me200. Login200 del resumen es literal; no captura status del login. No monta web-shell en navegador.
- **Límites de aceptación:** no se localizó prueba de rollback usuario+outbox ante fallo, broker/efecto consumidor, JWT en otro servicio ni exportación de spans/ingesta/alertas IAM. El compose completo con Tempo descrito en antecedentes no es resultado de esta corrida.
- **Otros servicios:** el E2E histórico notification con happy/DLQ/idempotencia conserva su expediente; no se reejecutó en esta entrega.

## Deuda de CI (nombrada)

Estado de los workflows consultados en ramas/PR IAM (sin fusión ni promoción acreditadas):

- Service:CI aprobada con186 pruebas;55 IT omitidos sin DB/broker. La evidencia PostgreSQL satisfactoria es local.
- DB:CI aprobada con PostgreSQL efímero, validate/migración/repetición/preflight.
- MFE:CI fallida en control de secreto de dependencias privadas; instalación/lint/tipos/tests/tokens/build no ejecutados allí.
- TresIT flaky y automatización del gate Pact son deuda histórica no cerrada por esta corrida. No se verificó enforcement de checks ni estado CI de otros servicios.

Deuda registrada: **HU-NOTIF-009 (docs#78)** para montar CI + Pact automatizado, enlazada con [TD-010](../15-project-control/technical-backlog.md) y [ci-cd.md](../10-devops/ci-cd.md).

## Trazabilidad y evidencia

- Cada HU probada se registra con la plantilla [_template-test-evidence.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/_template-test-evidence.md) (casos `TC-NNN`, defectos, trazabilidad `AC → HU → TC`).
- Cada sprint/release consolida resultados con [_template-qa-report.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/_template-qa-report.md), incluyendo el gate de calidad.
- [Expediente IAM](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md): [matriz por criterio](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) y [anexos saneados](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md), sin properties/system-out de los XML originales.
- La calidad forma parte de la [Definition of Done](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/definition-of-done.md): un cambio no está *done* si sus pruebas aplicables no pasan.

## Puntos abiertos

- Estandarizar utilería de contenedores efímeros por stack (Java y Go, ADR-006) para los servicios que aún no tienen capa de aplicación; `iam` y `notification` ya la resuelven a su manera.
- Umbral de cobertura y su automatización en CI: por cerrar con [ci-cd.md](../10-devops/ci-cd.md); ver **Deuda de CI** arriba (HU-NOTIF-009 / docs#78).
- Verificar deuda histórica de flakiness, automatizar Pact cuando corresponda y configurar CI MFE; no inferir su cierre por pruebas locales aprobadas.
