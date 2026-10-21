# DevOps

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Kevin Andrey Culma (Arquitecto) | Equipo: Arquitectura.

Índice de la sección de DevOps del sistema de Gestión de Horarios SENA. Cubre cómo se levanta el entorno local, cómo se validan y promueven los cambios, y qué ambientes existen.

## Estado real

La infraestructura documentada contempla PostgreSQL 16 compartido, perfiles de aplicación `iam` y `notification`, edge Traefik (ADR-007) y observabilidad (ADR-008). Los comandos de [local-setup.md](./local-setup.md) describen su preparación; esta revisión no certifica que los cuatro ambientes estén operativos ni que una ejecución histórica acredite la entrega IAM actual.

Al corte del 21 de octubre, las PR IAM [Service #43](https://github.com/code-sena/design-software-iam-service/pull/43), [DB #79](https://github.com/code-sena/design-software-iam-db/pull/79) y [MFE #10](https://github.com/code-sena/design-software-iam-mfe/pull/10) siguen abiertas, sin fusión. Service tiene CI aprobada con 186 casos y 55 IT omitidos; DB tiene CI aprobada con 37 changesets heredados desde cero, repetición sin cambios y preflight. MFE falla en CI por ausencia de `FRONTEND_DEPENDENCIES_READ_TOKEN`, antes de instalar o probar.

La validación local acredita 186 pruebas Service, 54 IT PostgreSQL aprobados y uno omitido, 46 casos MFE y un smoke HTTP del adaptador mediante proxy hacia API/DB. Usó una base aislada con su propietario; no acredita usuario de aplicación del entorno compartido, shell en navegador, consumidor RabbitMQ ni ingesta real de telemetría. El build Docker falló por DNS al descargar la imagen base. La promoción a `develop`, `qa`, `staging` o `main` de estas PR no está acreditada. El detalle y los anexos están en el [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

## Archivos

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [local-setup.md](./local-setup.md) | Levantar Postgres, aplicar changelogs por módulo y hacer bring-up de los perfiles de aplicación (`iam`, `notification`) + observabilidad en local | 🟡 |
| [ci-cd.md](./ci-cd.md) | CI verificada por componente IAM, límites de cobertura y controles/promoción pendientes | 🟡 |
| [environments.md](./environments.md) | Los 4 ambientes (`develop`/`qa`/`staging`/`main`), variables por `.env.<ambiente>` y advertencia de secretos | 🟡 |
| [runtime-configuration-map.md](./runtime-configuration-map.md) | Mapa de variables de configuración runtime detectadas por repositorio | 🟡 |

## Plantillas

| Plantilla | Descripción |
|-----------|-------------|
| [_template-deployment-plan.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/10-devops/_template-deployment-plan.md) | Plan de despliegue: componentes, pasos, verificación y rollback |
| [_template-release-checklist.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/10-devops/_template-release-checklist.md) | Checklist de release: gates de código, QA, seguridad e infraestructura |
| [_template-rollback-plan.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/10-devops/_template-rollback-plan.md) | Plan de rollback: criterios de activación y procedimientos por componente |

## Cómo se relaciona

- La **estrategia de migraciones** (orden, seeds idempotentes, rollbacks, tags) vive en [06-data/migration-strategy.md](../06-data/migration-strategy.md).
- La **topología de despliegue** (qué corre y dónde) está en [05-architecture/deployment.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/deployment.md).
- El **flujo de ramas y Conventional Commits** que gobierna la promoción está en [00-governance/git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md).
- El **riesgo de secretos versionados** en `.env.*` se detalla en [05-architecture/security-threat-model.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/security-threat-model.md) §Secretos.
