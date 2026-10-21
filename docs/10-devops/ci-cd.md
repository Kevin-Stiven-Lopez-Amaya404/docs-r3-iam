# CI/CD

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: DevOps

Pipeline de integración y entrega para el sistema de Gestión de Horarios SENA. Describe los controles automatizados sobre los cambios y cómo se promueven entre ambientes. Complementa a [00-governance/git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md), a [environments.md](./environments.md) y a [06-data/migration-strategy.md](../06-data/migration-strategy.md).

La disciplina de ramas, PR y Conventional Commits está definida en git-conventions. La automatización tiene avances verificables en las PR IAM, aún abiertas: no equivale a su fusión ni acredita enforcement de ramas protegidas en todos los repositorios. Los controles siguientes distinguen configuración, resultado observado y trabajo pendiente.

---

## Alcance

Los controles cubren tres capas que **ya existen**:

1. Validar que la **documentación** siga las convenciones (git-conventions, documentation-rules).
2. Validar que los **changelogs Liquibase** sean correctos y aplicables antes de promover.
3. Construir y probar la **capa de aplicación**: el Service IAM revisado registra 186 casos unitarios/MVC/contexto aprobados; la persistencia PostgreSQL tiene una ejecución local separada de 54 IT aprobados y uno omitido. Esta revisión no vuelve a validar los resultados de otros servicios.

## Ejecuciones IAM verificadas al 2026-10-21

| Componente y publicación | Ejecución | Resultado y límite |
|---|---|---|
| Service, [PR #43](https://github.com/code-sena/design-software-iam-service/pull/43), commit `7c837e0824263cef0aef89f807731adca839f25e` | [CI 39537700071](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071) | Java 21 / Maven 3.9.10, `verify`: 186 aprobados; 55 IT omitidos por no disponer de base/broker externos. Incluye contexto Spring; no acredita IT PostgreSQL remotos ni imagen Docker. |
| DB, [PR #79](https://github.com/code-sena/design-software-iam-db/pull/79), commit `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8` | [CI 39538261790](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) | PostgreSQL 16 / Liquibase 4.29: validate, migración limpia de 37 changesets existentes, segundo update sin cambios y preflight de esquema/permisos de grupos. No prueba rollback ni credenciales de la aplicación. |
| MFE, [PR #10](https://github.com/code-sena/design-software-iam-mfe/pull/10), commit `e73e844bebebb64b99bfce24a908926f0e4cf76b` | [CI 39540502857](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) | Fallida en el control de `FRONTEND_DEPENDENCIES_READ_TOKEN` ausente, antes de instalación, lint, tipos, tests y build. Los 46 casos y build aprobados son locales. |

Las tres PR permanecen abiertas, sin fusionar. El [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) relaciona commits, pruebas y anexos; no se ejecutaron pruebas nuevas para esta revisión documental.

---

## Controles del pipeline

| Control | Qué verifica | Estado |
|---------|--------------|--------|
| **Conventional Commits** | Formato `<type>(NN-section): description` en inglés; tipos permitidos (`docs`, `fix`, `chore`, `refactor`) | 🟡 Convención vigente; validación automática pendiente |
| **Lint de changelogs** | Estructura de cada changelog, `id` + `author` únicos por changeset, presencia de `rollback` espejo | 🟡 Regla definida ([migration-strategy.md](../06-data/migration-strategy.md)); linter pendiente |
| **Validación Liquibase** | Validate, update desde cero, repetición sin aumento de changesets y preflight IAM | 🟡 IAM DB CI aprobada en PR #79; otros repositorios y rollback no acreditados por esta revisión |
| **PR checks** | Revisión obligatoria antes de merge a rama protegida; sin commits directos a `develop`/`qa`/`staging`/`main` | 🟡 Reglas de rama vigentes; enforcement automático pendiente |
| **Seeds idempotentes** | Que los `02_dml` usen `ON CONFLICT DO NOTHING`/`MERGE` y que los datos de prueba estén aislados por `context`/`labels` | 🟡 Regla definida; verificación pendiente |
| **Secret scanning** | Que no se versionen credenciales reales en `.env.*` ni en el diff | 🔴 **Pendiente — riesgo real** (ver [security-threat-model.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/security-threat-model.md) §Secretos) |
| **Contract testing (Pact)** | Gate de compatibilidad de contratos entre consumidor y proveedor sobre `shared-contracts` (OpenAPI/eventos) | 🟡 Gate definido y ejecutable (skill de contratos); **corre a mano, no en CI** |
| **Build / tests de aplicación** | Compilación y pruebas por componente, separando unitarios/contexto de IT con infraestructura | 🟡 IAM Service CI: 186 aprobados y 55 IT omitidos; MFE CI bloqueada antes de probar; 54 IT PostgreSQL aprobados solo localmente |
| **`@SpringBootTest` de contexto completo** | Arranque del contexto Spring completo | 🟡 Incluido y aprobado en IAM Service PR #43; requisito de check bloqueante por servicio aún por confirmar |

---

## Flujo previsto por Pull Request

```
Rama hija (hu-NN-dev)  ──PR──▶  develop
        │
        ├─ check: Conventional Commit válido
        ├─ check: liquibase validate + status (BD efímera limpia)
        ├─ check: rollback espejo presente por changeset
        ├─ check: secret scanning (sin credenciales en el diff)
        └─ review humano aprobado
                     │
                     ▼  (todos los checks en verde)
                  merge a develop
```

El diagrama es el flujo objetivo. Los workflows IAM incorporados en estas PR se configuran para `develop`; no se verificó el mismo conjunto de checks en `qa`, `staging` o `main`.

---

## Promoción por ramas/ambientes

La promoción sigue **`develop → qa → staging → main`** ([git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md)):

| PR hacia | Origen | Migraciones | Endurecimiento |
|----------|--------|-------------|----------------|
| `develop` | ramas `hu-*-dev` | forward + rollback local | validación básica |
| `qa` | `hu-*-qa` (merge/cherry-pick desde develop) | **forward-only** | + validación funcional |
| `staging` | `hu-*-staging` (desde qa) | **forward-only** | + validación de preproducción |
| `main` | `release/<iteracion>` (desde staging) | **forward-only** | + tag de release (`04_tcl/`) |

- **Forward-only** en `qa`/`staging`/`main`: el pipeline no revierte migraciones en el flujo normal; el rollback es contingencia documentada.
- Cada release etiqueta una versión (`04_tcl/`) para permitir `rollback <tag>` como plan de contingencia.
- Los **datos de prueba no se promueven**: se aíslan con `context`/`labels` de Liquibase para que no lleguen a `staging`/`main`.

---

## Pendientes de automatización (priorizados)

Los antecedentes de *ambiguous mapping* justifican conservar un arranque de contexto completo en CI. La PR Service ya ejecuta esa cobertura; no debe seguir documentándose como inexistente. El estado histórico de tres IT inestables no se volvió a verificar: la ejecución PostgreSQL revisada terminó sin fallos, con una omisión por broker ausente. El pendiente verificable es automatizar los IT con sus dependencias y desbloquear MFE.

| # | Acción | Prioridad |
|---|--------|-----------|
| 1 | Secret scanning en cada PR y sacar `.env.*` del control de versiones | **Crítica** |
| 2 | Revisar/fusionar los workflows IAM; habilitar IT Service con PostgreSQL/broker y configurar la credencial de lectura de dependencias privadas MFE para obtener una ejecución completa | **Crítica** |
| 3 | Confirmar el contexto completo IAM como check bloqueante de rama y extender la regla a los demás servicios Java | **Crítica** |
| 4 | Gate de contratos (Pact) como check obligatorio en CI, no manual | Alta |
| 5 | Documentar cualquier caso inestable mediante reporte reproducible; no trasladar el conteo histórico de tres casos como fallo actual sin evidencia | Alta |
| 6 | Completar rollback y reaplicación de migraciones con evidencia; IAM DB ya valida/update/repite/preflight en CI, pero no ejecuta rollback | Alta |
| 7 | Linter de Conventional Commits como check obligatorio | Alta |
| 8 | Enforcement automático de ramas protegidas (no merge sin checks en verde) | Media |

### Regla: `@SpringBootTest` de contexto completo obligatorio por servicio Java

Cada servicio Java debe incluir al menos un **`@SpringBootTest` que levante el contexto Spring completo** (full-boot) y correr en CI como check bloqueante. Es el único tipo de prueba que caza fallos de arranque como el *ambiguous mapping* (mapeos ambiguos, beans duplicados, wiring cross-módulo roto); un `@WebMvcTest` acotado a un controller no los ve. Sin este guardián en el pipeline, un servicio puede mergear "verde" y no arrancar.

## Referencias

- [00-governance/git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md) — ramas, PRs, Conventional Commits
- [00-governance/documentation-rules.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/documentation-rules.md) — reglas de contenido
- [06-data/migration-strategy.md](../06-data/migration-strategy.md) — changelogs, seeds, rollbacks, tags
- [environments.md](./environments.md) — ambientes y `.env`
- [05-architecture/deployment.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/deployment.md) — topología de despliegue
