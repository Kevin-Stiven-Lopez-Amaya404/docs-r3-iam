# Registro de Riesgos

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Proyecto

Riesgos del proyecto **Horarios SENA**. Consolida riesgos técnicos, de alcance, de equipo, externos y de seguridad. Sigue el formato de [_template-risk-register.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/15-project-control/_template-risk-register.md). La revisión del 21 de octubre actualiza IAM a partir de sus PR y resultados; los riesgos de otros dominios conservan su alcance histórico y no se consideran reauditados. Ver [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).

## Cómo se calcula la exposición

**Exposición = Probabilidad × Impacto** (escala Baja=1 / Media=2 / Alta=3).

- Baja: producto < 3
- Media: producto 3–6
- Alta: producto > 6

## Riesgos activos

| ID | Categoría | Riesgo | Prob. | Impacto | Exposición | Mitigación | Owner | Estado |
|----|-----------|--------|-------|---------|------------|------------|-------|--------|
| R-001 | Seguridad | Secretos `.env.*` versionados en los repos `-db` pueden filtrar credenciales de base de datos. | Alta | Alto | Alta | Purgar del historial, rotar credenciales, mover a Secret Manager y bloquear vía `.gitignore`. Ver TD-001 en [technical-backlog.md](./technical-backlog.md). | PM/Arquitecto | Abierto |
| R-002 | Técnico | Doble anidamiento/rutas rotas se registraron en otros repos DB; no pueden asumirse corregidos globalmente. IAM ya aplica 37 changesets heredados desde cero y repite update en local/CI de PR #79. | Media | Alto | Media | Mantener validación por repo y revisar/fusionar CI IAM; no recrear su DDL ni extender el resultado a todos los módulos. Ver TD-002/TD-003. | Equipo Datos | Abierto; IAM mitigado en PR |
| R-003 | Alcance | El corte histórico de agosto registraba 2/10 servicios con aplicación; no se dispone de una auditoría actual de todos los servicios. Confundir ese dato o una PR IAM abierta con integración aceptada distorsiona el cronograma. | Media | Alto | Media | Mantener estado por HU y versión. IAM tiene Service/DB/MFE entregados en PR abiertas, con aceptación E2E pendiente; reauditar los demás dominios antes de actualizar el conteo global. | PM/Arquitecto | Abierto |
| R-005 | Técnico | La combinación IAM de las PR actuales carece de evidencia de broker/consumidor real y DLQ; los antecedentes de E2E de agosto no prueban esta entrega. | Media | Alto | Media | Disponer broker/consumidor, ejecutar relay y trazar contrato/consumo con versiones fijadas; conservar el IT omitido como pendiente. Ver TD-010. | Equipo IAM/Infra | Abierto para entrega actual |
| R-004 | Técnico | `scheduling-service` supera el límite de 2 dependencias síncronas (depende de actors, training-environment y academic-management). | Media | Alto | Media | Implementar read models según [ADR-002](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-002-scheduling-read-models.md); aprobar la ADR antes de construir el motor. | Arquitectura | Abierto |
| R-006 | Seguridad | Integridad de datos débil en `document-service` y `monitoring-service` (faltan CHECK e índices de FK) puede permitir estados inválidos y consultas costosas. | Media | Medio | Media | Reforzar esquema (TD-005/TD-006) y aplicar estados parametrizables de [ADR-004](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-004-status-parametrization-and-audit-standard.md) antes de exponer API. | Equipo Datos | Abierto |
| R-007 | Equipo | Concentración de conocimiento: el ownership y conocimiento de los **10 microservicios backend** puede concentrarse en pocas personas; ausencia o rotación frena todo el proyecto (factor bus = 1). | Media | Alto | Media | Documentar decisiones (ADRs, runbooks), mantener este repo de docs como fuente de verdad, incorporar y formar equipo por dominio. | PM | Abierto |
| R-008 | Técnico | Convención de ramas inconsistente entre repos rompe la trazabilidad `develop → qa → staging → main` y la promoción de cambios. | Media | Medio | Media | Homologar ramas protegidas y naming de [git-conventions.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/git-conventions.md); plantillas de PR comunes. Ver TD-007. | PM/Arquitecto | Abierto |
| R-009 | Externo | Driver JDBC obsoleto frente a PostgreSQL 16 puede causar incompatibilidades en despliegue de migraciones. | Baja | Medio | Baja | Actualizar y fijar versión de driver soportada (TD-004); validar en CI. | Equipo Datos | Abierto |
| R-010 | Alcance | Consumidores y proveedores pueden divergir del contrato. IAM MFE corrigió email/password y TokenResponse plano; el smoke real valida login/me, pero falta shell/edge del entorno objetivo y gate Pact completo. | Media | Medio | Media | Mantener contrato y versiones junto al servicio, pruebas del adaptador y aceptación integrada. No atribuir cambios OpenAPI a esta entrega. | Arquitectura | Abierto; adaptador IAM mitigado localmente |
| R-011 | Técnico | CI MFE no lee dependencias privadas por ausencia de `FRONTEND_DEPENDENCIES_READ_TOKEN`; falla antes de instalar/probar. | Alta | Alto | Alta | Configurar secreto con acceso de lectura a frontend-platform/design-system y obtener una ejecución completa; no exponerlo al navegador. Ver TD-013. | Mantenedor MFE/CI | Abierto |
| R-012 | Seguridad | Validación PostgreSQL con propietario de base aislada no acredita autenticación ni mínimos privilegios del usuario real. | Media | Alto | Media | Provisionar membresía de grupos y comprobar operaciones permitidas/denegadas con las credenciales de aplicación. El preflight solo valida grupos. Ver TD-014. | Equipo IAM/Infra | Abierto |
| R-013 | Técnico | Correlación HTTP y Prometheus local pueden confundirse con shell o telemetría distribuida aceptados. | Media | Alto | Media | Recoger evidencia propia de shell, spans OTLP/Tempo, scrape Prometheus, Loki y alertas; no reutilizar capturas de Monitoring como IAM. Ver TD-015. | Equipo IAM/Observabilidad | Abierto |
| R-014 | Externo | El build Docker Service falló por DNS al descargar `eclipse-temurin:21-jre`; no hay imagen nueva validada. | Media | Medio | Media | Resolver descarga, construir y probar imagen con commit identificado antes de declarar despliegue. Ver TD-016. | Equipo IAM/Infra | Abierto |

## Bloqueantes activos

| ID | Bloqueante | Condición de desbloqueo | Responsable |
|----|-----------|-------------------------|-------------|
| B-001 | Promoción segura de ambientes con secretos históricos pendientes de saneamiento; no invalida los builds locales ya registrados. | Tratar R-001 por ambiente; no publicar credenciales en anexos. | PM/Arquitecto |
| B-002 | Despliegue de esquema de repos no verificados; IAM DB ya tiene CI aprobada, aún en PR abierta. | Validación por módulo y revisión/fusión de PR #79; no exigir DDL nuevo para IAM. | Equipo Datos |
| B-003 | `scheduling-service` no puede diseñarse en detalle sin decisión de read models. | Aprobar ADR-002. | Arquitectura |
| B-004 | CI MFE detenida por credencial de dependencias ausente. | Configurar `FRONTEND_DEPENDENCIES_READ_TOKEN` con lectura y obtener CI completa. | Mantenedor MFE/CI |

## Riesgos aceptados

| ID | Riesgo | Justificación |
|----|--------|---------------|
| RA-001 | En el MVP, `scheduling-service` propaga el JWT de usuario a servicios downstream en lugar de usar tokens M2M. | Simplicidad de MVP; el scope del usuario aplica en toda la cadena. Migración a token M2M prevista en V2 (ver [authentication.md](../07-api/authentication.md)). |

## Histórico de riesgos declarados resueltos

| ID | Riesgo | Resolución | Estado |
|----|--------|------------|--------|
| R-005 (antecedente de agosto) | El registro del 28 de agosto declaraba broker/E2E y observabilidad operativos en `develop`. | Se conserva como antecedente de ese corte, sin extrapolarlo a los commits IAM de octubre. El ensayo actual omitió relay por broker ausente y no recogió ingesta de telemetría; el riesgo de integración de esta entrega figura arriba. | Histórico; resolución actual no acreditada |
