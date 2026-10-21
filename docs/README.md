# Índice de documentación IAM

> Estado: en revisión | Corte documental: 2026-10-21, America/Bogota

Esta carpeta reúne el expediente personal IAM y las copias de documentación
del macroproyecto adaptadas para reflejar la entrega revisada. Conserva la
jerarquía temática del repositorio oficial dentro de `docs/`; es una selección,
no una copia completa ni una actualización de su fuente oficial.

## Informe y evidencias de la entrega

| Documento | Lectura recomendada |
|---|---|
| [Expediente IAM](11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) | Alcance y resultados principales. |
| [Informe técnico](11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md) | Actividades y archivos realizados, versiones, pruebas, incidencias y límites. |
| [Matriz de evidencias](11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) | Qué cambio y criterio respalda cada prueba o anexo. |
| [Anexos revisados](11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md) | Evidencia histórica accesible y limitaciones de interpretación. |
| [Manifiesto](11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/manifest.json) | Procedencia, transformaciones y hashes de los anexos. |
| [Alineación documental](alineacion-documental.md) | Qué se adaptó, su origen y alcance respecto al macroproyecto. |

## Documentos adaptados por área

| Área | Documentos incluidos |
|---|---|
| Requisitos | [Trazabilidad](04-requirements/traceability-matrix.md), [historias](04-requirements/user-stories.md) y [requisitos no funcionales](04-requirements/non-functional.md). |
| Arquitectura | [Visión del sistema](05-architecture/overview.md). |
| Datos | [Estrategia de migraciones](06-data/migration-strategy.md) y [modelo IAM](09-microservices/services/01-iam-service/data-model.md). |
| API | [Índice de API](07-api/README.md), [autenticación](07-api/authentication.md) y [contrato narrativo IAM](09-microservices/services/01-iam-service/components/iam-api/contract.md). |
| Diagramas | [Índice](08-uml/diagram-index.md) y [secuencia de login](08-uml/diagrams/source/login-jwt.md). |
| Servicio IAM | [Descripción](09-microservices/services/01-iam-service/README.md), [runbook](09-microservices/services/01-iam-service/runbook.md), [decisiones](09-microservices/services/01-iam-service/decisions.md) y [componente API](09-microservices/services/01-iam-service/components/iam-api/README.md). |
| DevOps | [Índice](10-devops/README.md), [CI/CD](10-devops/ci-cd.md), [setup local](10-devops/local-setup.md), [ambientes](10-devops/environments.md) y [configuración](10-devops/runtime-configuration-map.md). |
| Calidad | [Índice](11-quality/README.md), [estrategia de pruebas](11-quality/testing-strategy.md) y [revisión de código](11-quality/code-review.md). |
| UX y MFE | [Índice](12-ux-ui/README.md), [navegación](12-ux-ui/navigation-map.md), [microfrontends](12-ux-ui/mockup-ddd/micro-frontends.md) y [flujo de autenticación](12-ux-ui/mockup-ddd/flows/01-auth.md). |
| Operaciones | [Índice](13-operations/README.md) y [observabilidad](13-operations/observability.md). |
| Control del proyecto | [Dependencias](15-project-control/dependencies.md), [riesgos](15-project-control/risks.md) y [backlog técnico](15-project-control/technical-backlog.md). |
| Registro de cambios | [Changelog documental](CHANGELOG.md). |

## Interpretación de los resultados

Se conservan **186 casos Service aprobados**, **54 IT PostgreSQL aprobadas y
una omitida**, y **46 casos MFE aprobados** localmente. CI Service y DB fueron
aprobadas en las PR revisadas; Service omitió las 55 IT externas en CI. CI MFE
falló por ausencia de la credencial para dependencias privadas, antes de
instalación/pruebas. Las PR Service #43, DB #79 y MFE #10 permanecían abiertas,
sin fusionar al corte. Los enlaces y commits están en el informe.

El smoke atraviesa adaptador/proxy/API/PostgreSQL; no demuestra montaje del
shell en navegador ni telemetría distribuida. El intento de build Docker falló.
Los procedimientos documentados y las tareas pendientes no se presentan como
pruebas ejecutadas o trabajo ya completado.

## Origen y autoría

Las copias parten de
[`design-software-docs` @ `917d7f6b514a4ff48d86f7ae78273f06e0df8a1c`](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c).
Los documentos conservan las autorías de origen. Esta selección no incorpora
el expediente Monitoring ni reutiliza sus evidencias como propias de IAM.
La cuenta publicadora de las PR IAM es `Kevin-Stiven-Lopez-Amaya404`; los
ejecutores de cada corrida requieren identificación mediante registros de los
participantes, sin inferirlos de esa cuenta.

La documentación oficial y su tracker siguen siendo las fuentes del
macroproyecto. Esta entrega personal no los actualiza ni certifica el cierre
del plan. [Volver al índice del repositorio](../README.md).
