# Backlog Técnico

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Arquitectura

Deuda técnica conocida y pendientes de construcción del proyecto **Horarios SENA**. La plataforma combina repositorios de datos y aplicaciones con madurez desigual; este backlog registra trabajo estructural pendiente sin asumir un único estado para todos los servicios.

Ver contexto en [service-catalog.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/service-catalog.md), [migration-strategy.md](../06-data/migration-strategy.md) y [dependency-map.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/09-microservices/dependency-map.md).

## Leyenda

- **Impacto**: Bajo / Medio / Alto — efecto sobre despliegue, integridad o velocidad del equipo.
- **Prioridad**: P0 (bloqueante) · P1 (alta) · P2 (media) · P3 (baja).
- **Estado**: 🔴 Abierto · 🟡 En curso · 🟢 Resuelto.

## Deuda técnica activa

| ID | Descripción | Impacto | Prioridad | Servicio(s) afectado(s) | Estado |
|----|-------------|---------|-----------|-------------------------|--------|
| TD-001 | **Secretos `.env.*` versionados**: los archivos de entorno (`.env.develop/.qa/.staging/.main`) con credenciales de conexión están bajo control de versiones. Requiere purga del historial, rotación de credenciales y mover secretos a un gestor (Secret Manager) referenciado desde `.gitignore`. | Alto | P0 | Todos los `*-db` | 🔴 |
| TD-002 | **Doble anidamiento en repos DB registrados históricamente**: verificar/corregir por módulo según [modeling-conventions §7](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/06-data/modeling-conventions.md#7-estructura-ddl-y-orden-de-aplicación-liquibase). IAM ya aplica su changelog sin reestructurar DDL en esta entrega. | Alto | P0 | Repos `*-db` no revalidados | 🔴; IAM comprobado en PR |
| TD-003 | **Reproducibilidad Liquibase por repositorio**: IAM aplica 37 changesets existentes desde cero, validate y update repetido en local/CI #79; falta revisión/fusión y no se prueba rollback. No extender ese resultado a otros módulos. | Alto | P0 | Varios `*-db` | 🟡; CI IAM aprobada en PR |
| TD-004 | **Driver JDBC de PostgreSQL obsoleto**: la versión de driver empaquetada con Liquibase está desactualizada frente a PostgreSQL 16. Actualizar a una versión soportada y fijarla explícitamente para reproducibilidad. | Medio | P1 | Todos los `*-db` | 🔴 |
| TD-005 | **Falta de índices en llaves foráneas**: las FK declaradas no tienen su índice de apoyo, en contra de la regla "un índice por cada FK" de [migration-strategy.md](../06-data/migration-strategy.md). Degrada joins y borrados en cascada. | Medio | P1 | `document-service`, `monitoring-service` | 🔴 |
| TD-006 | **`document-service` sin restricciones ni índices**: el esquema de `document` carece de `CHECK` de dominio (estados/tipos), de índices de consulta y de índices de FK. Reforzar antes de exponer la API y alinear con [ADR-003](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-003-object-storage.md) y [ADR-004](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-004-status-parametrization-and-audit-standard.md). | Medio | P1 | `document-service` | 🔴 |
| TD-007 | **Convención de ramas inconsistente entre repos**: no todos los repos `-db` siguen el flujo `develop → qa → staging → main` ni el naming `hu-<n>-dev` de [git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md). Homologar ramas protegidas y plantillas de PR. | Medio | P2 | Todos los `*-db` | 🔴 |
| TD-008 | **Madurez desigual de aplicaciones**: varios servicios siguen como skeleton/diseño parcial mientras otros ya poseen API/worker. Completar casos de uso y gates por servicio. | Alto | P1 | Varios servicios | 🟡 |
| TD-009 | **CHECK de estados aún no parametrizados de forma uniforme**: [ADR-004](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-004-status-parametrization-and-audit-standard.md) define estados parametrizables + auditoría estándar, pero no todos los servicios lo aplican todavía. Cerrar la brecha transversal. | Medio | P2 | `document`, `monitoring`, otros | 🔴 |
| TD-010 | **Validación del broker y contratos asíncronos pendiente por ambiente**: RabbitMQ es requerido por Audit, Monitoring y Notification; completar configuración, DLQ, retry y pruebas de contrato en CI. | Medio | P1 | `audit`, `monitoring`, `notification`, productores de eventos | 🟡 |
| TD-011 | **Completar separación contractual Monitoring → Notification**: retirar del `shared-contracts` canónico cualquier operación `/sent-notifications` o evento `monitoring.notification.sent`; mantener `notification.yaml` y `notification.notification.sent` como ownership de Notification. | Alto | P0 | `shared-contracts`, `monitoring`, `notification` | 🔴 |
| TD-012 | **Bandeja de notificaciones sin contrato de colección**: UX requiere `GET /notifications`, pero el OpenAPI vigente solo define creación y consulta por ID. Diseñar contrato paginado/cursor bajo Notification antes de implementar la campana global. | Medio | P1 | `notification`, `shell-host`, `shared-contracts` | 🔴 |
| TD-013 | **Desbloquear CI MFE**: configurar `FRONTEND_DEPENDENCIES_READ_TOKEN` con lectura de frontend-platform/design-system; la ejecución revisada falla antes de instalación, lint, tipos, tests y build. Obtener ejecución completa antes de cerrar el gate. | Alto | P1 | `iam-mfe`, mantenedor CI | 🔴 |
| TD-014 | **Usuario de aplicación y rollback verificables**: probar conexión/membresías/permisos con usuario real del entorno; anexar rollback conjunto usuario/outbox ante fallo y, si se declara, rollback/reaplicación de migraciones. Propietario local y ROLLBACK del preflight no acreditan estos criterios. | Alto | P1 | `iam-service`, `iam-db`, Infra | 🔴 |
| TD-015 | **Aceptación IAM en el macroproyecto**: montar shell en navegador, validar rutas edge, consumidor/broker, spans OTLP/Tempo, scrape Prometheus, Loki y alertas con versiones fijadas. Separar de smoke del adaptador y de evidencia Monitoring de otra autoría. | Alto | P1 | IAM, shell, consumidores, Observabilidad | 🔴 |
| TD-016 | **Imagen y promoción de la combinación IAM**: repetir build Docker tras resolver DNS de descarga, validar imagen y revisar/fusionar PR Service #43/DB #79/MFE #10; acreditar despliegue por ambiente sin deducirlo de CI. | Alto | P1 | IAM, Infra, revisores | 🔴 |

## Avance verificable del plan IAM

Al corte 2026-10-21, las tres PR permanecen abiertas, sin fusionar. Se entregaron correlación/métricas y CI Service; preflight/CI/documentación DB sin DDL nuevo; y adaptador MFE alineado al contrato, puertos e inyección, proxy y CI configurada. Los resultados locales son 186 pruebas Service, 54 IT PostgreSQL aprobados y uno omitido, 46 casos MFE, build federado y smoke HTTP adaptador→API/DB.

CI Service aprobó 186 casos y omitió 55 IT; CI DB migró 37 changesets heredados desde cero, repitió sin cambios y aprobó preflight; CI MFE falló por secreto ausente antes de probar. No se da por resuelta flakiness histórica, gate Pact global ni integración completa por estos resultados. Actividades, autoría y anexos: [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

## Trabajo estructural pendiente (capa de aplicación)

Desglose de TD-008 en fases relativas. No se comprometen fechas hasta cerrar los bloqueantes P0.

> **Corte histórico del 2026-08-28:** se registraban 2/10 servicios con aplicación (`iam-service` y `notification-service`). No se reauditaron los demás servicios para esta actualización; ese conteo no representa un inventario actual. La revisión IAM de octubre acredita los avances anteriores y mantiene pendiente su aceptación integrada. Consultar estado por HU/versiones y no equiparar el antecedente broker/E2E con la combinación de las PR actuales.

| Fase | Alcance | Depende de | Estado |
|------|---------|------------|--------|
| Fase A — Fundaciones de datos | Cerrar TD-001…TD-006; base de datos reproducible y segura por servicio. | — | 🟡 En curso |
| Fase B — Núcleo de identidad | IAM API: login, JWT RS256, RBAC por feature+scope; entrega de mejora Service/DB/MFE. | Fase A | 🟡 API existente con regresiones locales aprobadas; mejora en PR abiertas, integración/promoción no acreditadas |
| Fase C — Catálogos y dominio base | `reference-data-service`, `academic-management-service`, `training-environment-service`, `actors-service`. | Fase B | Corte de agosto: pendiente; estado actual no reauditado |
| Fase D — Motor de horarios | `scheduling-service` + read models de [ADR-002](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-002-scheduling-read-models.md). | Fase C | Corte de agosto: pendiente; estado actual no reauditado |
| Fase E — Transversales | `document-service`, `audit-service`, `monitoring-service` y `notification-service`; validar separación por eventos según ADR-005. | Fase B–D | Corte de agosto: parcial con Notification; estado actual no reauditado |

## Criterio de ingreso al backlog

Un ítem entra aquí cuando: (1) está identificado con servicio y evidencia concreta, (2) tiene impacto y prioridad asignados, y (3) no es una tarea de contenido puramente documental (esas van a las ramas `docs/*`). Al resolverse se marca 🟢 y se mueve al histórico en el cierre de iteración.
