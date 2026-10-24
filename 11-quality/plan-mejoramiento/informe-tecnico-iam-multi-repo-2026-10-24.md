# Informe técnico de desarrollo — alineación multi-repositorio IAM

> **Estado:** En revisión, con validaciones pendientes
>
> **Fecha de corte:** 24 de octubre de 2026, 
>
> **Responsable:** Kevin Stiven López Amaya
>
> **Plan individual:** R3 — observabilidad en IAM Service
>
> **Tipo de documento:** Informe técnico de desarrollo e integración; no es un informe de aprobación QA.

> **Nota sobre la plantilla:** en los archivos disponibles del repositorio y en la carpeta de descargas consultada no se encontró una plantilla identificada como “Plantilla de desarrollador”. Este informe usa una estructura técnica de desarrollo con identificación, alcance, ramas, modificaciones, pruebas, evidencias, dependencias y pendientes. Si existe una plantilla distinta, puede aplicarse después sin alterar los resultados aquí registrados.

## 1. Resumen ejecutivo

Se revisaron y documentaron las ramas de IAM Service, IAM DB e IAM MFE solicitadas para mantenerlas alineadas con sus respectivas ramas `develop`. La comparación remota confirma que las tres ramas de trabajo tienen **cero commits detrás** de su referencia. Los PR #43, #79 y #10 continúan abiertos y dirigidos a `develop`.

Service y DB aprobaron sus workflows en las puntas actuales. La CI de MFE se detuvo antes de instalar dependencias porque falta el secreto `FRONTEND_DEPENDENCIES_READ_TOKEN`; por tanto, esa ejecución no probó lint, tipos, pruebas ni build. La alineación de historial está comprobada, pero todavía no se acredita una prueba integrada con las tres puntas actuales ni el cierre completo de QA.

La asignación individual de Kevin es R3 de observabilidad en **IAM Service**. DB y MFE se incluyen en este informe como trabajo adicional de alineación solicitado para apoyar la integración del proyecto; no se presentan como repositorios formalmente asignados en el plan individual.

## 2. Alcance y referencias

Se tomó `develop` de cada repositorio como rama de referencia. Los SHA y conteos siguientes fueron contrastados con la comparación remota de GitHub al corte del informe.

| Repositorio | Rama de trabajo | Rama de referencia y SHA | Punta revisada | Diferencia remota | PR |
|---|---|---|---|---:|---|
| [IAM Service](https://github.com/code-sena/design-software-iam-service) | `r3-observability-delivery-dev` | `develop` · `c7735cfc3135aad83091e2cfac5abe5809853c85` | `06ab67bd54c0a561e20dde784c0191d67df30e6c` | 4 adelante / **0 detrás** | [#43 abierta](https://github.com/code-sena/design-software-iam-service/pull/43) |
| [IAM DB](https://github.com/code-sena/design-software-iam-db) | `r3-iam-db-integration-dev` | `develop` · `a1e01a42692da77269568f0ef49b2e4d3d035fa3` | `b89574a7eac05e2edef4d8d3f61df08f40214ce7` | 2 adelante / **0 detrás** | [#79 abierta](https://github.com/code-sena/design-software-iam-db/pull/79) |
| [IAM MFE](https://github.com/code-sena/design-software-iam-mfe) | `r3-iam-mfe-integration-dev` | `develop` · `8e94e1bd281ca12010fac754e7c376f2d078c20f` | `301cf6c4f5d07ddf2677bf3d4198ea8481e2720e` | 3 adelante / **0 detrás** | [#10 abierta](https://github.com/code-sena/design-software-iam-mfe/pull/10) |

Los tres PR permanecen abiertos. No se renombraron ni eliminaron ramas, no se cerraron ni reemplazaron PR, y las actualizaciones publicadas avanzaron las ramas sin reescribir su historial.

## 3. Trabajo realizado por repositorio

### 3.1 IAM Service — R3 de observabilidad y alineación con `develop`

- **Repositorio y rama:** [IAM Service / `r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev).
- **PR:** [#43](https://github.com/code-sena/design-software-iam-service/pull/43).
- **Commit documental más reciente:** [`06ab67b`](https://github.com/code-sena/design-software-iam-service/commit/06ab67bd54c0a561e20dde784c0191d67df30e6c).

**Cambios funcionales presentes en la rama:**

- `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/CorrelationIdFilter.java`: valida el encabezado `X-Correlation-Id`, genera un UUID cuando falta o no cumple el patrón esperado, lo devuelve en la respuesta y restaura el MDC al finalizar la solicitud.
- `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/ApiErrors.java`: prioriza el `trace_id` activo de OpenTelemetry en la respuesta de error, con los fallbacks existentes.
- `iam-api/pom.xml` y `iam-api/src/main/resources/application.yml`: configuración de Micrometer/Prometheus y métricas HTTP con etiqueta de servicio, contadores e histogramas de duración.
- La rama incorpora también los controles de sesión y autorización que ya están en `develop`, conservando las pruebas y automatización R3.

**Documentación de Service:** se encuentran el registro de decisión `docs/observability/ADR-009-r3-observability.md`, la guía `docs/observability/integration.md` y `docs/observability/validation.md`. El último ajuste documental actualiza la referencia `develop`, enlaza la punta revisada y deja visible el desfase del contrato de revocación. No se editó manualmente la copia OpenAPI fijada.

**Pruebas y estado:**

- [Workflow IAM build and tests, run 40092215283](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215283): exitoso en la punta revisada.
- [Workflow de controles negativos R4, run 40092215448](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215448): exitoso en la punta revisada.
- El registro de validación documenta 201 pruebas aprobadas y 20 controles negativos R4 aprobados; 56 pruebas que requieren DB o broker quedaron omitidas en esa ejecución.
- El smoke PostgreSQL previamente documentado usó Service `7c837e0824263cef0aef89f807731adca839f25e` con DB `develop@a1e01a42692da77269568f0ef49b2e4d3d035fa3`. Es evidencia histórica y no valida la combinación actual de puntas.

**Sin cambios en esta entrega:** no se añadieron migraciones de base de datos ni se afirma la integración con un despliegue institucional.

### 3.2 IAM DB — preflight y validación de migraciones existentes

- **Repositorio y rama:** [IAM DB / `r3-iam-db-integration-dev`](https://github.com/code-sena/design-software-iam-db/tree/r3-iam-db-integration-dev).
- **PR:** [#79](https://github.com/code-sena/design-software-iam-db/pull/79).
- **Commit documental más reciente:** [`b89574a`](https://github.com/code-sena/design-software-iam-db/commit/b89574a7eac05e2edef4d8d3f61df08f40214ce7).

**Cambios presentes en la rama:**

- `scripts/verify-iam-integration.sql`: preflight de solo lectura para revisar tablas, columnas, índices y privilegios de los grupos que usa IAM Service. El script termina con `ROLLBACK` y no crea ni altera el esquema.
- `.github/workflows/ci.yml`: ejecuta validación Liquibase, migración sobre PostgreSQL 16, repetición de la actualización para comprobar idempotencia y el preflight de integración.
- `docs/integration/iam-service.md` y `README.md`: describen requisitos, comandos y procedencia de las pruebas. El ajuste más reciente aclara que la prueba de Service conservada es histórica y no representa el candidato actual.

**Pruebas y estado:**

- [Workflow DB, run 40092216422](https://github.com/code-sena/design-software-iam-db/actions/runs/40092216422): exitoso en la punta revisada; el job migró una base limpia, verificó la actualización incremental y ejecutó el preflight.
- La evidencia existente registra 37 changesets del changelog, validación correcta y una segunda actualización sin cambios nuevos.
- Una ejecución anterior de integración registró pruebas PostgreSQL de Service, pero utilizó el commit histórico de Service indicado en la sección anterior. No debe presentarse como prueba de compatibilidad con la punta actual.

**Sin cambios en esta entrega:** la rama no introduce DDL ni changesets nuevos. El preflight tampoco aprovisiona ni autentica el usuario real de despliegue.

### 3.3 IAM MFE — cliente de autenticación y coordinación con el host

- **Repositorio y rama:** [IAM MFE / `r3-iam-mfe-integration-dev`](https://github.com/code-sena/design-software-iam-mfe/tree/r3-iam-mfe-integration-dev).
- **PR:** [#10](https://github.com/code-sena/design-software-iam-mfe/pull/10).
- **Commit documental más reciente:** [`301cf6c`](https://github.com/code-sena/design-software-iam-mfe/commit/301cf6c4f5d07ddf2677bf3d4198ea8481e2720e).

**Cambios funcionales presentes en la rama:**

- `src/adapter/out/auth.ts`: adapta login a `POST /api/v1/auth/login` con `email` y `password`, valida la respuesta contractual del token, maneja errores y respuestas vacías de recuperación de contraseña y añade `X-Correlation-Id`.
- `src/application/ports/AuthenticationPort.ts`, `src/application/authentication.tsx`, `src/contracts/remote-props.ts` y `src/App.tsx`: permiten inyectar el transporte del host y entregar la sesión mediante `onAuthenticated`. El MFE no toma propiedad de la persistencia de sesión del shell.
- `vite.config.ts`: configura el proxy local `/api/v1` y la exposición del adaptador para el host.
- `tests/unit/authAdapter.test.ts` y `tests/integration/PasswordResetFlow.test.tsx`: agregan regresiones para el contrato de autenticación y recuperación.
- `README.md` y `docs/integration/iam-service.md`: explican la fuente del contrato, dependencias y evidencia. El documento aclara que los tipos se generan actualmente desde una copia en `design-software-docs` y no se deben regenerar desde `shared-contracts` hasta resolver el desfase.

**Pruebas y estado:**

- La validación local registrada —lint, typecheck, 46 pruebas y revisión de tokens— se ejecutó sobre el código funcional `e73e844bebebb64b99bfce24a908926f0e4cf76b`; la punta actual añade documentación.
- [CI MFE, run 40092215804](https://github.com/code-sena/design-software-iam-mfe/actions/runs/40092215804): falló en la comprobación de acceso a dependencias privadas. Instalación, pruebas y build quedaron omitidos.
- El intento de build local temporal emitió `remoteEntry.js`, pero el plugin de tipos registró `EPERM` al escribir a través de un `node_modules` enlazado y de solo lectura. Por ello, un build limpio y aislado sigue sin confirmarse.
- El smoke MFE → API → PostgreSQL previo corresponde a MFE `e73e844`, Service `7c837e0` y DB `develop@a1e01a4`; no valida las tres puntas actuales.

**Dependencia pendiente:** un administrador debe configurar `FRONTEND_DEPENDENCIES_READ_TOKEN` con permiso de lectura para `design-software-design-system` y `design-software-frontend-platform`, y volver a ejecutar CI. El token no debe guardarse en Git ni escribirse en el PR.

## 4. Convenciones y dependencias entre repositorios

| Tema | Convención o dependencia verificada | Estado |
|---|---|---|
| Ramas | Las tres ramas de entrega siguen sus respectivos `develop`; en la comparación actual ninguna está detrás. | Verificado al corte indicado. |
| API IAM | MFE envía `email/password` al prefijo `/api/v1` y valida el `TokenResponse` generado desde OpenAPI. | Código y pruebas locales disponibles; smoke actual conjunto pendiente. |
| Contrato canónico | Service declara que su copia OpenAPI se obtiene de `design-software-shared-contracts`. La copia fijada de Service marca `IDENTITY_USER_VIEW` para revocar una sesión ajena; el código requiere `IDENTITY_USER_MANAGE` y permite autogestión. | Diferencia confirmada; la política esperada requiere decisión del responsable del contrato. |
| Tipos MFE | `generate:iam-types` lee actualmente `../design-software-docs/07-api/contracts/openapi/iam.yaml`; los tipos generados no se editan a mano. | No cambiar a la versión actual de `shared-contracts` sin revisar las diferencias y regenerar de forma coordinada. |
| DB | Service consume un esquema existente y las pruebas de integración requieren PostgreSQL. El preflight de DB es de solo lectura. | CI de DB aprobada; smoke contra las puntas actuales pendiente. |
| Broker | Los escenarios del relay/outbox requieren broker disponible. | Pruebas dependientes omitidas donde no se configuró RabbitMQ. |
| Dependencias privadas MFE | CI necesita leer dos repositorios hermanos privados mediante `FRONTEND_DEPENDENCIES_READ_TOKEN`. | Bloquea actualmente el workflow de MFE. |
| Identidad de cambios documentales | GitHub muestra autor y committer `Kevin-Stiven-Lopez-Amaya404` en los commits documentales más recientes de Service, DB y MFE. | Verificado en GitHub. |

## 5. Evidencias principales

- [Comparación Service `develop...r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/compare/develop...r3-observability-delivery-dev).
- [Comparación DB `develop...r3-iam-db-integration-dev`](https://github.com/code-sena/design-software-iam-db/compare/develop...r3-iam-db-integration-dev).
- [Comparación MFE `develop...r3-iam-mfe-integration-dev`](https://github.com/code-sena/design-software-iam-mfe/compare/develop...r3-iam-mfe-integration-dev).
- [Informe QA actualizado, sin HU](revision-qa-alineacion-iam-sin-hu-2026-10-23.md).
- [Registro de validación del servicio](https://github.com/code-sena/design-software-iam-service/blob/r3-observability-delivery-dev/docs/observability/validation.md).
- [Guía de integración de DB](https://github.com/code-sena/design-software-iam-db/blob/r3-iam-db-integration-dev/docs/integration/iam-service.md).
- [Guía de integración de MFE](https://github.com/code-sena/design-software-iam-mfe/blob/r3-iam-mfe-integration-dev/docs/integration/iam-service.md).
- CI: [Service](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215283), [controles R4](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215448), [DB](https://github.com/code-sena/design-software-iam-db/actions/runs/40092216422), [MFE bloqueada](https://github.com/code-sena/design-software-iam-mfe/actions/runs/40092215804).

## 6. Pendientes para completar la validación

1. Configurar el secreto de lectura privado de MFE mediante un administrador y volver a ejecutar el workflow en la punta actual.
2. Acordar con el responsable de `design-software-shared-contracts` si la autorización correcta para revocar sesiones ajenas es `VIEW` o `MANAGE`. Después, actualizar la fuente canónica, sincronizar el pin de Service y regenerar los tipos dependientes de MFE. No elegir el permiso por inferencia.
3. Ejecutar un smoke nuevo con Service `06ab67bd54c0a561e20dde784c0191d67df30e6c`, DB `b89574a7eac05e2edef4d8d3f61df08f40214ce7` y MFE `301cf6c4f5d07ddf2677bf3d4198ea8481e2720e`, usando PostgreSQL de pruebas y RabbitMQ donde aplique. Conservar comandos, resultados y SHA de cada componente.
4. Confirmar `npm ci`, lint, typecheck, pruebas y build federado de MFE en un checkout limpio con dependencias privadas disponibles.
5. Validar MFE montado en el shell real y aportar evidencia de métricas, logs, trazas y alertas si esos componentes están incluidos en los criterios de entrega.
6. Registrar la revisión técnica y el resultado final de cada PR. La comparación de ramas y los checks verdes no sustituyen una revisión aprobatoria.

## 7. Conclusión técnica

Al corte del informe, las ramas de Service, DB y MFE están alineadas por historial con sus referencias y conservan los PR originales abiertos. Service y DB aprobaron CI en sus puntas actuales. MFE tiene un bloqueo externo de acceso a dependencias; el contrato de permisos y las pruebas integradas con los SHA actuales siguen pendientes. Por eso, el resultado se describe como **alineación de ramas comprobada con validación integral incompleta**, no como entrega libre de observaciones ni aprobación final de QA o producción.

La documentación de cada repositorio registra alcance y procedencia: [Service `06ab67b`](https://github.com/code-sena/design-software-iam-service/commit/06ab67bd54c0a561e20dde784c0191d67df30e6c), [DB `b89574a`](https://github.com/code-sena/design-software-iam-db/commit/b89574a7eac05e2edef4d8d3f61df08f40214ce7) y [MFE `301cf6c`](https://github.com/code-sena/design-software-iam-mfe/commit/301cf6c4f5d07ddf2677bf3d4198ea8481e2720e). El informe QA complementario está en [`revision-qa-alineacion-iam-sin-hu-2026-10-23.md`](revision-qa-alineacion-iam-sin-hu-2026-10-23.md).
