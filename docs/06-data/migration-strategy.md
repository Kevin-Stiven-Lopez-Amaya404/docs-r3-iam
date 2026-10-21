# Estrategia de migraciones (Liquibase)

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Datos

Estándar transversal de cómo se versiona y despliega el esquema de base de datos de cada servicio. Complementa a [modeling-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/06-data/modeling-conventions.md) (§7 estructura DDL) y a [local-setup.md](../10-devops/local-setup.md).

## Herramienta y unidad de despliegue

- **Liquibase** sobre **PostgreSQL 16**. Cada servicio se despliega desde su repositorio `*-db`.
- Cada repo tiene un **changelog maestro** (`changelog-master.yaml`) que incluye, en orden, los changelogs por carpeta descritos en [modeling-conventions §7](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/06-data/modeling-conventions.md#7-estructura-ddl-y-orden-de-aplicación-liquibase).
- Cada changeset declara `id` + `author` únicos y define su `rollback` (espejo en `05_rollbacks/`).

### Aplicación verificada en IAM

Al corte del 21 de octubre de 2026, la [PR DB #79](https://github.com/code-sena/design-software-iam-db/pull/79) permanece abierta, commit `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8`, sobre la base `a1e01a42692da77269568f0ef49b2e4d3d035fa3`. Añade preflight de solo lectura, CI y documentación; **no agrega ni modifica DDL o changesets**. El punto de entrada utilizado es `changelog/changelog-master.yaml`.

La migración local desde cero ejecutó 37 changesets heredados; validate aprobó y un segundo update ejecutó 0 nuevos, manteniendo 37 previos. El [workflow DB aprobado](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) reproduce validación, migración limpia, repetición sin aumento y preflight en PostgreSQL 16 efímero con Liquibase 4.29. La repetición no prueba un changeset incremental nuevo, rollback ni reaplicación posterior al rollback.

`identity.outbox`, índice y grants ya existían en el [commit 5dd269c](https://github.com/code-sena/design-software-iam-db/commit/5dd269c28e4487ca69f4722480920f5aa81512a2). Las columnas temporales de `identity.user` y el índice de recuperación proceden de la [PR #74](https://github.com/code-sena/design-software-iam-db/pull/74), fusionada el 25 de agosto; se conservan su autoría e historial. El fallo inicial por outbox ausente correspondió a una demo incompleta, no a ausencia de esas migraciones en el repositorio oficial.

El preflight verifica esquema y privilegios de grupos `NOLOGIN`; no crea/autentica al usuario de aplicación ni prueba sus permisos efectivos. Se ensayó con propietario de base aislada, sin modificar la compartida. El `ROLLBACK` que termina su transacción de lectura tampoco demuestra reversión de migraciones o rollback conjunto de usuario/outbox ante fallo de negocio. Esos resultados y límites se trazan en el [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

## Aislamiento por módulo

- Todos los módulos conviven en **una base de datos**, pero **cada módulo crea y usa su propio schema** (`identity`/`rbac`/`session`, `academic_management`, `training_environment`, `monitoring`, `institutional_structure`, `document`, etc.). Ningún módulo escribe en `public`.
- Se recomienda **aislar el tracking de Liquibase por módulo** (`--liquibase-schema-name=<modulo>`), de modo que cada servicio tenga su propia `databasechangelog` y su historial sea auditable y reversible de forma independiente.

## Orden de aplicación (por qué importa)

1. `00_extensions` → `01_schemas` → `02_types`
2. `03_tables` — **tablas sin llaves foráneas**
3. `04_alter` — **llaves foráneas vía `ALTER TABLE ... ADD CONSTRAINT`, después de crear toda la estructura** (evita rupturas por orden de creación / referencias hacia adelante)
4. `05_views` … `08_triggers` → `10_indexes` (incluye un índice por cada FK)
5. `02_dml` (seeds) → `03_dcl` (roles/grants) → `04_tcl` (tags de versión)

## Datos semilla (seeds)

- Van en `02_dml/` y deben ser **idempotentes**: usar `INSERT ... ON CONFLICT DO NOTHING` (o `MERGE`) para poder re-ejecutarse sin fallar.
- Separar **datos de catálogo** (semilla real de negocio) de **datos de prueba** (que no deben ir a `staging`/`main`); estos últimos se aíslan con `context`/`labels` de Liquibase.

## Rollbacks

- Todo changeset forward tiene su rollback espejo en `05_rollbacks/` con la **misma ruta relativa**.
- Rollback de `04_alter` → `DROP CONSTRAINT`; de `03_tables` → `DROP TABLE`; de `02_dml` → `DELETE`/inverso.
- Los rollbacks de datos deben revertir primero los datos y luego el constraint (p. ej. `UPDATE` de valores antes de recrear un `CHECK`).

## Versionado y ambientes

- **Tags de release** en `04_tcl/` al cierre de cada iteración, para poder hacer `rollback <tag>`.
- Promoción por ambientes: `develop → qa → staging → main` (ver [git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md)). Cada ambiente tiene su propio archivo de entorno (`.env.develop/.qa/.staging/.main`).
- **En `qa`/`staging`/`main` las migraciones son forward-only**; los rollbacks se usan en desarrollo local y como plan de contingencia documentado, no como flujo normal en ambientes compartidos.

## Lógica almacenada

- Funciones, vistas y triggers (`05_views`/`06_functions`/`07_procedures`/`08_triggers`) deben marcarse con **`runOnChange: true`** para que Liquibase re-aplique el objeto cuando cambie su definición.
