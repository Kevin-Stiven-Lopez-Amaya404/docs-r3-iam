# Matriz de cambios, criterios y evidencias IAM

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Cuenta publicadora de las PR de implementación: Kevin-Stiven-Lopez-Amaya404 | Equipo: IAM

## Contexto

Relaciona las actividades S/D/F y pruebas T/C del [informe](./informe-tecnico.md) con las fuentes conservadas. `B1/B2/B3` son bloques de [HU-INT-IAM-001/docs#290](https://github.com/code-sena/design-software-docs/issues/290), no nuevos IDs oficiales de historias o un acta firmada del plan. El ejecutor de cada corrida requiere bitácora individual; la cuenta autora Git acredita implementación/publicación.

La matriz y los anexos revisados se preparan para publicación en el repositorio personal
`Kevin-Stiven-Lopez-Amaya404/docs-r3-iam`. La [alineación documental](../../../alineacion-documental.md)
identifica las copias adaptadas de la documentación oficial y su procedencia. Publicarlas aquí
no actualiza la fuente oficial ni fusiona las PR de Service, DB o MFE. Los anexos incluidos
conservan extractos y resultados transformados; no se publican los logs ni XML originales
completos. La cuenta publicadora no permite inferir quién ejecutó cada prueba.

## Contenido

### Cambio → prueba → evidencia

| Cambio realizado | Archivo/caso concreto | Prueba y resultado | Evidencia preparada para esta entrega |
|---|---|---|---|
| S01 correlación HTTP | `CorrelationIdFilterTest` (6 casos), `HealthControllerTest` (2 MVC) | T01 aprobada; T07 devuelve correlación. | [Casos XML transformados](./evidencias/service-test-results.json), [resumen runtime](./evidencias/runtime-smoke-summary.json). |
| S02 trace ID del error | `ApiErrorsTest` (3 casos) | T01 aprobada; no prueba exportación de spans. | JSON de casos y [resumen Maven](./evidencias/service-tests.txt). |
| S03 métricas | `IamApiApplicationContextTest` (1 caso nuevo) | T01/T07: contador, buckets, etiqueta y endpoint. | Mismos anexos; [application.yml](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/iam-api/src/main/resources/application.yml). |
| S04/S05 build, CI y documentación | Wrapper, workflow, dockerignore y guías | T02 éxito; T08 fallo DNS; C01 aprobada con IT omitidos. | [Docker](./evidencias/docker-build-failure.txt), [consulta CI/PR](./evidencias/remote-status.json), enlaces del informe. |
| D01/D04 compatibilidad y preparación | Preflight SQL y changelog oficial sin DDL nuevo | T03/T04:37 y luego0 nuevos; T05 positivo y rechazo esperado; T06:54+1 omitida. | [DB](./evidencias/db-validation.txt), [PostgreSQL](./evidencias/postgres-tests.txt), JSON de casos. |
| D02/D03 automatización/guía | Workflow y `docs/integration/iam-service.md` | C02 aprobada: DB efímera, repeat update y preflight. | Consulta CI/PR y [código SQL](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/scripts/verify-iam-integration.sql). |
| F01/F02 contrato y recuperación | `authAdapter.test.ts`, `PasswordResetFlow.test.tsx` | T10:46 aprobadas totales, cuatro casos nuevos del adaptador. | [MFE](./evidencias/mfe-tests-build.txt), [adaptador](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/src/adapter/out/auth.ts). |
| F03 entrega de sesión/UI | Puerto/contexto, LoginScreen, callbacks/fixtures | T10 verifica comportamiento con mocks/Testing Library. | Mismo anexo MFE y código de PR10; no prueba shell real. |
| F04 proxy/transporte/exposición | `vite.config.ts`, adaptador y variables | T11 build; T12 sesión válida, me200, identidad/correlación correctas. | [Smoke](./evidencias/http-smoke-summary.json), [driver](./evidencias/smoke-http-adapter.mjs.txt). Login200 en el resumen es literal. |
| F05/F06 CI y documentación | Workflow, README y guía | C03 falla en secreto; pasos posteriores omitidos. T09 aprobación tipos/lint declarada con logs incompletos. | Consulta CI/PR, anexo MFE; no adjudicar CI verde. |
| F07 fixture/persistencia/limpieza | Driver PowerShell y material efímero Java | Driver contiene aserción SQL1\|1\|1 y cleanup; no stdout SQL histórico separado. | [Driver](./evidencias/smoke-http-driver.ps1.txt), [Java](./evidencias/smoke-fixture.java.txt), [INSERT/DELETE](./evidencias/mfe-fixture-cleanup.txt). |

### Criterio HU#290 → caso → alcance

| Criterio | Caso/fuente disponible | Resultado que puede afirmarse |
|---|---|---|
| B1 base limpia con esquema requerido | T03–T05, C02 | 37 changesets heredados y preflight comprobados. PR79 no crea outbox/columnas; el diagnóstico “objetos faltantes” debe referirse a la demo incompleta. |
| B1 arranque con validate | Configuración existente + T07 | JAR operativo contra esquema compatible aislado, sin generación automática de DDL. |
| B1 auditoría de login | `LoginPostgresIT#loginSucceeds` aprobado | Login200 y auditoría SUCCESS consultada en PostgreSQL. |
| B1 refresh persistido sin token plano | Mismo caso + `SecureRandomRefreshTokenGenerator` | Una sesión, hash no nulo/distinto del token; SHA256 por inspección del generador, no comparación explícita de digest en ese test. |
| B1 estado del usuario | `loginSucceeds`, `successResetsTheCounter`, `loginWorksAfterTheLockElapsed` aprobados | last_login_at presente, failed_attempts0, locked_untilNULL; cambio de dos fallos a cero comprobado. |
| B1 creación y evento | `UserManagementPostgresIT#createUserReturnsTemporaryPasswordAndStagedEvent` aprobado | HTTP201 y evento iam.user.created relacionado. No consulta fila usuario directamente en ese caso; otras regresiones comprueban lectura/estado. |
| B1 atomicidad y rollback conjunto ante fallo | UserApplicationService `@Transactional`; CreateUser save+stage; adaptadores JDBC | Frontera implementada por inspección; **no se localizó prueba de rollback conjunto**. duplicateEmailAnswers409 falla antes de guardar. |
| B2 JWT en otros servicios, scopes/centro/rotación | Sin ejecución específica conservada | No acreditado por auth/me del propio IAM. |
| B3 navegador/shell/edge/broker/consumidor | T10–T12, T08, C03 | Adaptador/proxy/API/DB local acreditados; recorrido completo y montaje real pendientes de evidencia. |
| B3 reporte de auditoría autorizado y trazas | T06 fila audit; T07 health/ready/métricas | Esas pruebas no acreditan consulta del reporte por usuario autorizado ni trazas distribuidas. |

Los casos PostgreSQL anteriores son regresiones existentes, no pruebas nuevas del plan. El JSON permite ubicar clase/método y estado sin publicar propiedades ni salidas sensibles de los XML originales.

### Calidad de la evidencia y responsables

| Tipo | Identificación | Limitación |
|---|---|---|
| Implementación/publicación | Commits y PR de cuenta Kevin-Stiven-Lopez-Amaya404 | No identifica ejecutor de todas las pruebas ni distribución entre participantes. |
| Ejecución local | Logs/extractos y XML históricos | Completar bitácora individual; no se reejecutaron al preparar este expediente. |
| Declaración con respaldo parcial | Tipos/lint, SQL del smoke, cleanup | Guía y código disponibles; exit codes completos y stdout SQL por tabla no conservados. |
| Estado remoto | Snapshot con fecha y enlaces CI/PR | Corte mutable; no certifica fusión, promoción o checks obligatorios. |
| Documentación personal | Copias adaptadas y anexos incluidos en `docs-r3-iam` | Se publicarán con esta entrega; no sustituyen ni modifican la fuente oficial y mantienen las autorías históricas. |

## Referencias

- [Índice del expediente](./README.md), [Informe](./informe-tecnico.md), [Anexos/manifiesto](./evidencias/README.md).
- [Copia adaptada de trazabilidad](../../../04-requirements/traceability-matrix.md), [HU#290](https://github.com/code-sena/design-software-docs/issues/290).
- [Alineación documental y procedencia de las copias](../../../alineacion-documental.md).
