# Ambientes

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: DevOps

Los ambientes del sistema de Gestión de Horarios SENA, cómo se configuran y qué reglas rigen en cada uno. Complementa a [00-governance/git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md), a [local-setup.md](./local-setup.md) y a [06-data/migration-strategy.md](../06-data/migration-strategy.md).

## Los cuatro ambientes

El flujo de gobierno define una rama por ambiente, recepción de cambios por PR y protección de ramas. Esta revisión no audita el enforcement de cada repositorio ni deduce un despliegue de la existencia de una rama.

| Ambiente | Rama | Propósito | Base de datos | Migraciones |
|----------|------|-----------|---------------|-------------|
| **develop** | `develop` | Integración del trabajo en curso | BD de desarrollo | Forward + rollback local permitido |
| **qa** | `qa` | Validación funcional y técnica | BD de QA | **Forward-only** |
| **staging** | `staging` | Preproducción / validación previa al release | BD de preproducción | **Forward-only** |
| **main** | `main` | Producción / documentación estable | BD de producción | **Forward-only** |

Promoción: **`develop → qa → staging → main`**. El detalle del flujo de ramas (ramas `hu-*` por ambiente, `release/*` hacia `main`) está en [git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md).

### Estado verificado de IAM al 2026-10-21

Las PR Service #43, DB #79 y MFE #10 apuntan a `develop` y siguen abiertas, sin fusionar. La validación disponible corresponde a una base local aislada y a CI efímera; no certifica despliegue de esos cambios en la base compartida ni promoción a `qa`, `staging` o `main`. Los perfiles IAM/Notification, edge Traefik y stack OTel descritos en la infraestructura son configuración disponible y antecedentes, no resultados nuevos de este plan.

El ensayo local utilizó PostgreSQL en `55432` (`iam_delivery_20261021`), JAR directo `18080` y proxy Vite `15101`; no montó shell en navegador. Las credenciales fueron del propietario de la base aislada. Faltan usuario de aplicación real, consumidores y telemetría distribuida del entorno objetivo; el build Docker falló por DNS. Resultados y referencias de publicación: [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

---

## Puertos expuestos al host por ambiente (ADR-007)

La tabla recoge puertos de la configuración de infraestructura para coexistencia local. No prueba que los recursos estén vivos en esos ambientes. Los puertos del ensayo IAM aislado se indican arriba y no cambian esta asignación de Compose.

| Recurso | develop | qa | staging | main |
|---|---|---|---|---|
| Edge (Traefik) — único entry point | 18080 | 18081 | 18082 | 18083 |
| Traefik dashboard | 18090 | 18091 | 18092 | 18093 |
| Postgres | 15432 | 15433 | 15434 | 15435 |
| RabbitMQ mgmt | 15672 | 15673 | 15674 | 15675 |
| Grafana / Tempo / Prometheus / Loki (ADR-008) | base + offset | +1 | +2 | +3 |
| `iam-api` (y cualquier `-api` de dominio) | **no expuesto** — solo vía edge | | | |

El detalle operativo de bring-up de estos puertos vive en [local-setup.md](./local-setup.md).

---

## Configuración por ambiente — `.env.<ambiente>`

La configuración de cada ambiente vive en un archivo de entorno propio:

| Archivo | Ambiente |
|---------|----------|
| `.env.develop` | develop |
| `.env.qa` | qa |
| `.env.staging` | staging |
| `.env.main` | main |

Docker Compose lee `.env` por defecto, por lo que el archivo se pasa **explícitamente** en cada comando:

```bash
docker compose --env-file .env.develop up postgres -d
docker compose --env-file .env.develop --profile tooling run --rm liquibase-<modulo> update
```

Estos archivos parametrizan la conexión a Postgres (host, puerto, nombre de BD, usuario, clave) del ambiente correspondiente y, además, la **configuración de la capa de aplicación** de los perfiles que corren en ese ambiente:

- **Datos:** conexión a Postgres del ambiente.
- **Aplicación (`iam`):** `IAM_DB_DSN`, `IAM_DB_USERNAME` / `IAM_DB_PASSWORD` (usuario de aplicación, ver `create-app-user.sh`), `IAM_JWT_PRIVATE_KEY` / `IAM_JWT_ISSUER` (y demás `IAM_JWT_*`).
- **MFE:** `VITE_IAM_API_URL` según base API publicada; `IAM_API_PROXY_TARGET` para el proxy de desarrollo. El secreto CI `FRONTEND_DEPENDENCIES_READ_TOKEN` solo permite leer dependencias privadas y no se incluye en el bundle.
- **Edge / infra:** `EDGE_PORT` (puerto del Traefik del ambiente), `BACKEND_REPOS_DIR` (carpeta de repos de backend clonados para el build context).

El detalle operativo de los comandos y la tabla completa de variables por servicio están en [local-setup.md](./local-setup.md).

---

## Base de datos por ambiente

- Cada ambiente tiene su **propia base de datos**, con la misma estructura: **una BD, un schema por módulo** (`identity`/`rbac`/`session`, `academic_management`, `training_environment`, `monitoring`, `institutional_structure`, `document`, `audit`, `scheduling`, …). Ningún módulo escribe en `public`.
- Para la base compartida se recomienda aislar **tracking de Liquibase por módulo** (`--liquibase-schema-name=<modulo>`); verificar la configuración real antes de aplicar. El ensayo IAM usó una base exclusiva y no acredita que ese aislamiento esté configurado en todos los ambientes.
- **Datos por ambiente:**
  - `develop` admite **datos de prueba** (seeds con `context`/`labels` de Liquibase).
  - `qa`/`staging`/`main` reciben **solo catálogos de negocio** (semillas reales, idempotentes con `ON CONFLICT DO NOTHING`); los datos de prueba se aíslan para que **no** se promuevan.
- En `qa`/`staging`/`main` las migraciones son **forward-only**; el rollback es plan de contingencia documentado, apoyado en los **tags de release** (`04_tcl/`), no operación rutinaria.

---

## Advertencia de secretos

> **Riesgo real y prioritario.** Los archivos `.env.*` **no deben contener contraseñas reales versionadas en git**. Si una credencial de base de datos queda en el repositorio (o en su historial), cualquiera con acceso al repo obtiene acceso a los datos del ambiente, incluida PII sujeta a la Ley 1581/2012.

Reglas:

- Versionar únicamente un **`.env.example`** con valores ficticios como plantilla; mantener los valores reales **fuera de git**.
- Ante una credencial que haya estado versionada: **rotar primero, avisar después** ([security-rules.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/security-rules.md)), sacar el archivo del control de versiones y evaluar limpieza del historial.
- Estado objetivo: mover los secretos a un **Secret Manager**; la existencia de aplicaciones o de secretos CI no acredita que esa migración esté completada.

Detalle en [05-architecture/security-threat-model.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/security-threat-model.md) §Secretos y nota de secretos en [local-setup.md](./local-setup.md).

## Referencias

- [local-setup.md](./local-setup.md) — comandos por ambiente
- [ci-cd.md](./ci-cd.md) — validaciones IAM comprobadas y promoción/controles pendientes
- [00-governance/git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md) — ramas protegidas y flujo de promoción
- [06-data/migration-strategy.md](../06-data/migration-strategy.md) — seeds, contexts, rollbacks, tags
- [05-architecture/deployment.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/deployment.md) — topología de despliegue por ambiente
