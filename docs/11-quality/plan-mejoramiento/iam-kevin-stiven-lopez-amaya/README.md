# Plan de mejoramiento IAM: entrega y evidencias

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Cuenta publicadora de las PR de implementación: Kevin-Stiven-Lopez-Amaya404 | Equipo: IAM

## Contexto

Expediente personal preparado para `Kevin-Stiven-Lopez-Amaya404/docs-r3-iam`, con los cambios realizados en IAM Service, DB y MFE y las pruebas históricas disponibles. Se revisaron código, diferencias respecto a `develop`, PR, CI, logs y reportes locales; preparar estos anexos no implicó ejecutar nuevamente las pruebas.

Los documentos bajo `docs/` son copias adaptadas de `code-sena/design-software-docs@917d7f6b514a4ff48d86f7ae78273f06e0df8a1c` o archivos nuevos de este expediente. Su alineación y publicación aquí no modifican la fuente oficial, no fusionan las PR de implementación ni certifican aceptación. La [alineación documental](../../../alineacion-documental.md) identifica alcance y procedencia; se conservan las autorías históricas de las fuentes.

Los extractos, resúmenes, resultados transformados y drivers revisados incluidos en `evidencias/` se publicarán con esta entrega. No se incluyen los logs ni XML originales completos: el manifiesto conserva su procedencia y distingue los hashes de las copias preparadas de los originales locales.

Este expediente es independiente del de [Monitoring / Kevin Andrey Culma Gómez](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/11-quality/plan-mejoramiento/r3-observabilidad/README.md). No incorpora sus capturas ni resultados como evidencia IAM. No se localizó el acta original firmada del plan individual; la publicación de las PR identifica una cuenta autora, pero no permite completar el reparto de ejecución entre personas.

## Contenido

| Documento | Propósito |
|---|---|
| [Informe técnico](./informe-tecnico.md) | Actividades realizadas, versiones, pruebas, incidencias, sincronización documental y límites. |
| [Matriz de evidencias](./matriz-evidencias.md) | Relación cambio → criterio HU → prueba → resultado → anexo. |
| [Anexos y manifiesto](./evidencias/README.md) | Extractos de logs, resultados XML transformados, resúmenes, drivers conservados y procedencia/hash. |
| [Alineación documental de esta entrega](../../../alineacion-documental.md) | Copias adaptadas en el repositorio personal y relación con la fuente oficial. |

### Resultado documentado al 21 de octubre

| Componente | Resultado local | Publicación y CI consultadas |
|---|---|---|
| Service | 186 pruebas aprobadas; PostgreSQL: 54 aprobadas y una omitida. JAR con health/ready/Prometheus disponibles. | [PR43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta; [CI](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071) aprobada, con 55 IT omitidos allí. |
| DB | 37 changesets desde cero; validate, segundo update sin cambios y preflight positivo/negativo comprobados. | [PR79](https://github.com/code-sena/design-software-iam-db/pull/79) abierta; [CI](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) aprobada. No se añadió DDL. |
| MFE | 46 pruebas aprobadas, build federado y smoke HTTP del adaptador/proxy/API/DB. | [PR10](https://github.com/code-sena/design-software-iam-mfe/pull/10) abierta; [CI](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) fallida por credencial de dependencias privadas ausente. |

Las tres PR continúan sin fusión en la [consulta conservada](./evidencias/remote-status.json). El estado vivo sigue en GitHub; esta tabla es un corte documental. Los cambios se describen como entrega en rama, sin certificación de promoción o cierre de HU-INT-IAM-001.

El smoke acredita sesión de login válida, identidad coincidente, correlación y `/auth/me`200. El valor `login:200` del resumen es un literal del driver, no captura de su status HTTP. No se acredita montaje del web-shell en navegador, consumidores, broker ni telemetría distribuida IAM. El build Docker falló por DNS. La frontera transaccional usuario/outbox existe en código; no se localizó prueba de rollback conjunto ante fallo.

## Referencias

- [HU-INT-IAM-001 / docs#290](https://github.com/code-sena/design-software-docs/issues/290).
- [Copia adaptada de trazabilidad](../../../04-requirements/traceability-matrix.md).
- [Definition of Done](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/definition-of-done.md).
