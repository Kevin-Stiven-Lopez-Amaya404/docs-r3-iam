# Calidad

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Calidad

## Contexto

Esta sección define cómo se asegura la calidad del sistema **Horarios SENA**. El corte histórico de agosto registraba aplicación construida en `iam-service` y `notification-service` y una fase de datos para los demás servicios. Ese inventario no constituye una auditoría actual del resto del macroproyecto. El stack es políglota Java + Go ([ADR-006](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-006-polyglot-stack-strategy.md)).

La actualización IAM del 21 de octubre distingue datos, aplicación backend, MFE e integración transversal. Su [expediente](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) registra 186 pruebas Service aprobadas, 54 PostgreSQL aprobadas y una omitida, 46 MFE aprobadas y smoke adaptador/proxy/API/DB. Service/DB tienen CI aprobada en sus PR; MFE está bloqueada por acceso privado. No se acredita aquí broker, shell real, consumidores o telemetría distribuida. Los antecedentes de [notification](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/services/10-notification-service/README.md) conservan su alcance y no se revalidaron en esta entrega.

Las prácticas aquí descritas se apoyan en la gobernanza ya definida: la [Definition of Done](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/definition-of-done.md), la [Definition of Ready](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/definition-of-ready.md), las [convenciones de Git](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md) (git flow `develop → qa → staging → main` y Conventional Commits) y la [estrategia de migraciones](../06-data/migration-strategy.md).

## Contenido

Define prácticas de pruebas, revisión de código y criterios de calidad.

## Archivos

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [testing-strategy.md](./testing-strategy.md) | Estrategia de pruebas por nivel y tipo | 🟡 |
| [code-review.md](./code-review.md) | Criterios y flujo para revisiones de código | 🟡 |
| [plan-mejoramiento/README.md](./plan-mejoramiento/README.md) | Índice de expedientes individuales IAM y Monitoring | 🟡 |

## Plantillas

| Plantilla | Descripción |
|-----------|-------------|
| [_template-qa-report.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/_template-qa-report.md) | Reporte de QA: cobertura, defectos, HUs verificadas y gate de calidad |
| [_template-test-evidence.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/_template-test-evidence.md) | Evidencia de pruebas: casos TC-NNN, defectos y trazabilidad AC-HU-TC |

## Puntos abiertos

- **CI por componente:** Service 186 aprobadas/55 IT omitidos; DB validate/migración/repetición/preflight aprobados; MFE falla antes de instalar/probar por secreto privado ausente. Son workflows en PR abiertas, no prueba de fusión, despliegue o checks obligatorios. Véanse [CI/CD](../10-devops/ci-cd.md) y [anexos](./plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md).
- La deuda histórica de tres IT flaky y el gate Pact (HU-NOTIF-009/docs#78, [TD-010](../15-project-control/technical-backlog.md)) no se cierra mediante una corrida satisfactoria de este plan.
- Umbral de cobertura exacto y su gate automatizado en CI: pendiente de cerrar junto con [ci-cd.md](../10-devops/ci-cd.md).
- Selección del set de prueba para los servicios que **aún no** tienen capa de aplicación: se decidirá por stack (Java o Go, según ADR-006) al iniciar su construcción.

- [Plan R3 de Monitoring](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/plan-mejoramiento/r3-observabilidad/README.md), con autoría/evidencias independientes de IAM.
