# Operaciones

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Operaciones

## Contexto

Describe cómo operar, monitorear, responder incidentes y recuperar el sistema
**Horarios SENA**, con alcance y evidencia por servicio y versión. Además de
PostgreSQL/Liquibase existe capa de aplicación IAM; su entrega R3 añade
correlación y métricas y conserva validaciones locales contra PostgreSQL.
Las PR Service #43, DB #79 y MFE #10 permanecían abiertas al corte; estos
resultados no acreditan despliegue compartido ni aceptación del plan.

El stack OpenTelemetry y LGTM está decidido en
[ADR-008](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/decisions/records/ADR-008-observability-opentelemetry.md).
La [evidencia IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md)
acredita HTTP, correlación y exposición de métricas, pero no scrape externo,
trazas en Tempo, ingesta Loki ni alertas IAM. El expediente Monitoring tiene su
propia autoría, fecha y recorrido; sus resultados no se transfieren a IAM.

## Contenido

Describe cómo operar, monitorear, responder incidentes y recuperar el sistema.

## Archivos

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [observability.md](./observability.md) | Métricas, logs, trazas, alertas y tableros | 🟡 |
| [incident-management.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/incident-management.md) | Clasificación, respuesta y comunicación de incidentes | 🟡 |
| [backup-and-recovery.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/backup-and-recovery.md) | Backups, restauración, RPO/RTO y pruebas de recuperación | 🟡 |

## Plantillas

| Plantilla | Descripción |
|-----------|-------------|
| [_template-runbook.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-runbook.md) | Runbook operativo: alertas, diagnóstico, procedimientos y escalamiento |
| [_template-observability.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-observability.md) | Observabilidad: logs, métricas RED/USE, trazas, alertas y healthchecks |
| [_template-sla-slo-sli.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-sla-slo-sli.md) | SLA/SLO/SLI: objetivos por servicio, error budget y políticas |
| [_template-incident-postmortem.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-incident-postmortem.md) | Postmortem: línea de tiempo, causa raíz, impacto y acciones correctivas |

## Puntos abiertos

- Validar y conservar evidencia del recorrido IAM en el stack ya seleccionado:
  shell, consumidores, scrape, Tempo, Loki, dashboard y alerta con recuperación.
- Completar integración bajo el usuario real de aplicación y construir la imagen
  IAM; el intento Docker conservado falló por DNS antes de compilar.
- SLO/SLA por servicio con números comprometidos: por definir cuando exista tráfico real (usar [_template-sla-slo-sli.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/13-operations/_template-sla-slo-sli.md)).
- Roles de guardia (on-call) y canales reales de escalamiento: por completar.
