# Informe verificable del plan de mejoramiento IAM

> Estado: 🟡 En revisión | Fecha de corte: 2026-10-21, America/Bogota
> Elaboración: revisión documental asistida con Codex | Equipo analizado: IAM
> Cuenta autora de las PR y commits de entrega revisados: `Kevin-Stiven-Lopez-Amaya404`.

## Contexto

Este informe organiza el trabajo comprobable en **IAM Service, IAM DB e IAM MFE**, sus pruebas conservadas y la documentación del macroproyecto y de los repositorios personales. Se contrastaron archivos, diferencias Git, reportes XML, logs, manifiestos y metadatos actuales de GitHub. Las pruebas de aplicación descritas son ejecuciones anteriores: **no se volvieron a ejecutar al elaborar este documento**.

Hay cambios implementados y publicados en ramas de entrega, resultados locales satisfactorios y documentación preparada. Esto respalda un informe de avance, pero **no acredita la aceptación completa del plan, la fusión de las entregas ni un despliegue integrado**. En la consulta del 21 de octubre, las tres PR seguían abiertas y sin fusionar; la HU de integración también estaba abierta.

El Acta 02 y el Anexo de Aporte Individual permiten identificar la asignación R3 de Kevin a IAM Service. La cuenta Git identifica autoría de commits y publicación de PR, pero no demuestra por sí sola quién ejecutó cada prueba local; la asignación de una HU tampoco identifica automáticamente al autor de todos los cambios relacionados.

### Enlaces y alcance de la asignación individual (actualización al 2026-10-23)

El **Acta 02 del Plan de Mejoramiento de Producto** y el **Anexo de Aporte Individual** revisados identifican a **Kevin Stiven López Amaya** con la asignación **R3 de observabilidad en IAM Service**. Por tanto, el repositorio de trabajo individual es [IAM Service, rama `r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev). El análisis de IAM DB y IAM MFE incluido en este informe documenta el contexto y la integración disponible; no implica que esos repositorios fueran asignados individualmente a Kevin.

Repositorios documentales relacionados:

- [Repositorio personal `docs-r3-iam`, rama `docs/doc-import-workspace`](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam/tree/docs/doc-import-workspace), donde se publicó esta documentación.
- [Repositorio documental del macroproyecto `design-software-docs`](https://github.com/code-sena/design-software-docs/tree/main/11-quality/plan-mejoramiento), ubicación institucional de seguimiento.

Los documentos fuente de la asignación fueron `ADSO-3145555-Acta02-Plan-Mejoramiento-Producto.pdf` y `ADSO-3145555-Anexo-Aporte-Individual.pdf`, proporcionados para revisión y no incluidos en este repositorio. Una persona revisora necesitará consultarlos por separado para verificar la asignación formal.

## Contenido

### 1. Interpretación del alcance y rutas de los repositorios

Se interpreta **“MF” como el microfrontend IAM MFE**, y **“micro” como IAM Service**. Los nombres `desing-software-...` se resuelven, por contexto y remotos Git, como `design-software-docs` y la entrega personal `docs-r3-iam`. Esta última sí contiene `docs/`; el repositorio documental del macroproyecto organiza sus secciones numeradas directamente en la raíz.

Las siguientes rutas fueron accesibles durante la revisión. Se usan separadores `/` para facilitar su lectura en Markdown.

| ID | Repositorio o copia | Ruta local comprobada | Remoto y función |
|---|---|---|---|
| R01 | Documentación del espacio actual | `C:/Users/PC/Desktop/design-software-docs` | [Oficial](https://github.com/code-sena/design-software-docs); remoto adicional `iam`: [docs-r3-iam personal](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam). Rama local `docs/doc-import-workspace`. Aquí se entrega este informe. |
| R02 | Service de entrega | `C:/Users/PC/Desktop/design-software-iam-service/iam-integration` | [code-sena/design-software-iam-service](https://github.com/code-sena/design-software-iam-service). Código y pruebas de PR43. |
| R03 | DB de entrega | `C:/Users/PC/Desktop/design-software-iam-service/iam-db-integration` | [code-sena/design-software-iam-db](https://github.com/code-sena/design-software-iam-db/tree/r3-iam-db-integration-dev). Rama solicitada explícitamente. |
| R04 | MFE de entrega | `C:/Users/PC/Desktop/design-software-iam-service/iam-mfe-integration` | [code-sena/design-software-iam-mfe](https://github.com/code-sena/design-software-iam-mfe). Código y pruebas de PR10. |
| R05 | Clon documental con alineación previa | `C:/Users/PC/Desktop/design-software-iam-service/design-software-docs` | Remoto oficial `code-sena/design-software-docs`; rama local `docs/iam-improvement-alignment`. Contiene cambios documentales locales y anexos. |
| R06 | Entrega documental personal | `C:/Users/PC/Desktop/design-software-iam-service/docs-r3-iam` | [Kevin-Stiven-Lopez-Amaya404/docs-r3-iam](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam). Contiene la selección documental en `docs/`, todavía no versionada en el estado local observado. |
| R07 | Repositorio personal Service que contiene los clones | `C:/Users/PC/Desktop/design-software-iam-service` | [Kevin-Stiven-Lopez-Amaya404/design-software-iam-service](https://github.com/Kevin-Stiven-Lopez-Amaya404/design-software-iam-service). Rama `main`, HEAD `97239a8`; los clones anidados y el análisis anterior aparecen sin seguimiento en ese repositorio. |
| R08 | Preparación anterior de Service | `C:/Users/PC/Desktop/iam-service-plan-mejoramiento` | Remoto oficial IAM Service; rama `r3-observability-dev`, HEAD `c9a02b6`. Conserva modificaciones locales en `docs/observability/`. Es antecedente, no la versión final PR43. |
| R09 | Copia anterior de trabajo Service | `C:/Users/PC/Desktop/design-software-iam-service/r3-service-worktree` | Remoto local hacia R08; HEAD `a765dd8`. Tiene cambios documentales sin confirmar. No es un remoto oficial independiente. |
| R10 | Dependencia frontend compartida | `C:/Users/PC/Desktop/design-software-iam-service/design-software-frontend-platform` | [code-sena/design-software-frontend-platform](https://github.com/code-sena/design-software-frontend-platform), HEAD `be8bd79fa0e5e33955c481d928c3867077ff851e`. Dependencia utilizada; sin cambios locales observados. |
| R11 | Dependencia de diseño compartida | `C:/Users/PC/Desktop/design-software-iam-service/design-software-design-system` | [code-sena/design-software-design-system](https://github.com/code-sena/design-software-design-system), HEAD `b1c8b763b6f812173a11b232e35fed15b4c2f62b`. Dependencia utilizada; sin cambios locales observados. |

**Rutas `docs/` identificadas:** R02: `docs/observability/`; R03 y R04: `docs/integration/`; R06: `docs/` con documentación adaptada del macroproyecto. La raíz R07 no tiene una carpeta `docs/` propia en el estado observado. La carpeta con ese nombre de R06 no debe confundirse con la estructura raíz de R01 o R05.

El HEAD de R01 y R05 es `556a779855b76a7db4bce35262c9ab10d99c4d58`. Ambos tienen el árbol base `885ce4415042b9ef319ed6e199cbceb890710176`, coincidente con el árbol de procedencia declarado en la alineación documental para el [baseline oficial `917d7f6b514a4ff48d86f7ae78273f06e0df8a1c`](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c). La coincidencia de contenido **no convierte los distintos historiales en el mismo commit ni prueba sincronización con el último `main` remoto**.

### 2. Versiones y publicación comprobadas

Para Service y MFE, cuyas URL recibidas no especifican rama, se revisaron las ramas de entrega asociadas con las PR y las evidencias locales. Se contrastaron contra las bases que declara cada PR, sin asumir que los cambios ya estén en `develop`.

| Componente | Base de comparación | Rama y commit de entrega | Publicación al corte |
|---|---|---|---|
| Service | `6987e0fa64628b5602969572d34a900f022ae891` | `r3-observability-delivery-dev` · `7c837e0824263cef0aef89f807731adca839f25e` | [PR43](https://github.com/code-sena/design-software-iam-service/pull/43), abierta y sin fusionar hacia `develop`. **15 archivos** cambiados. |
| DB | `a1e01a42692da77269568f0ef49b2e4d3d035fa3` | `r3-iam-db-integration-dev` · `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8` | [PR79](https://github.com/code-sena/design-software-iam-db/pull/79), abierta y sin fusionar hacia `develop`. **4 archivos** cambiados. |
| MFE | `8e94e1bd281ca12010fac754e7c376f2d078c20f` | `r3-iam-mfe-integration-dev` · `e73e844bebebb64b99bfce24a908926f0e4cf76b` | [PR10](https://github.com/code-sena/design-software-iam-mfe/pull/10), abierta y sin fusionar hacia `develop`. **17 archivos** cambiados, dos commits: funcional `01482e9` y CI `e73e844`. |

Los conteos se comprobaron con diferencias Git locales y los SHA se contrastaron con GitHub. Los tres clones de entrega estaban limpios. Las modificaciones de las copias anteriores R08/R09 y las del repositorio personal R07 se distinguen de esas entregas publicadas.

### 3. Actividades realizadas con evidencia verificable

Los identificadores **E01–E09** remiten al inventario de evidencias de la sección 5. Las rutas de código de cada fila son relativas al repositorio R indicado y a su commit de entrega de la sección 2.

#### 3.1. Cambios directos en IAM Service

| ID | Actividad comprobada | Modificación y ubicación | Evidencia y límite |
|---|---|---|---|
| S01 | Implementar correlación de solicitudes HTTP | R02, `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/CorrelationIdFilter.java`: acepta `X-Correlation-Id` con patrón `[A-Za-z0-9._-]{1,128}`, genera UUID si falta o es inválido, devuelve la cabecera y restaura MDC incluso ante excepción. | [Código fijado al commit](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/CorrelationIdFilter.java); `CorrelationIdFilterTest.java`, `HealthControllerTest.java`; E01/E05. No demuestra exportación de trazas. |
| S02 | Priorizar el identificador de traza activo en los errores | R02, `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/ApiErrors.java`: toma `trace_id` de MDC; después `X-Request-Id` y UUID. | `ApiErrorsTest.java`, tres casos aprobados en E01. Un UUID alternativo no acredita una traza consultable en Tempo. |
| S03 | Exponer métricas HTTP para Prometheus | R02, `iam-api/pom.xml` y `iam-api/src/main/resources/application.yml`: registro Prometheus, exposición Actuator, etiqueta `service` e histograma HTTP. | `IamApiApplicationContextTest.java`, E01/E05; endpoint y series presentes en el smoke local. Scrape y dashboard externos sin acreditar. |
| S04 | Preparar construcción y CI del servicio | R02, `.mvn/wrapper/maven-wrapper.properties`, `.github/workflows/ci.yml`, `.dockerignore`: Maven 3.9.10, Java 21, `verify` y carga de reportes. | E01 y [CI Service](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071). El build JAR pasó; el build Docker de E08 falló. |
| S05 | Documentar integración, decisión y validación | R02, `README.md`, `docs/observability/ADR-009-r3-observability.md`, `integration.md`, `validation.md`. | Archivos presentes en PR43. Documentan resultados locales y límites; no constituyen aceptación de la HU. |

El agente Java OpenTelemetry y la configuración de logs JSON ya tenían antecedentes en el servicio: **no se atribuye su creación a este diff**. Tampoco se presenta la lógica previa de negocio o las sondas existentes como desarrollo nuevo del plan.

#### 3.2. Cambios y validación de IAM DB

| ID | Actividad comprobada | Modificación y ubicación | Evidencia y límite |
|---|---|---|---|
| D01 | Implementar un preflight de compatibilidad con Service | R03, `scripts/verify-iam-integration.sql`: verifica tablas, columnas temporales, índices, seeds y permisos de grupos `iam_reader`, `iam_writer`, `iam_admin`; transacción de solo lectura terminada en `ROLLBACK`. | [Script fijado al commit](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/scripts/verify-iam-integration.sql), E04. El `ROLLBACK` de lectura no es una prueba de reversión de migraciones. |
| D02 | Automatizar comprobación de migración y compatibilidad | R03, `.github/workflows/ci.yml`: PostgreSQL 16, Liquibase 4.29, `validate`, `update`, segundo `update` y comparación del conteo, más preflight. | [CI DB](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) aprobada. No levanta el MFE, API ni broker. |
| D03 | Documentar la integración de datos | R03, `README.md`, `docs/integration/iam-service.md`: versión compatible, requisitos y procedimiento. | Archivos incluidos en PR79 y resultados E04. Los comandos de una guía no prueban por sí solos un despliegue completo. |
| D04 | Validar una base local nueva con el changelog oficial | Ejecución conservada sobre `iam_delivery_20261021`, en contenedor aislado `iam-r3-demo-20261020`: 37 changesets desde cero, validación y repetición sin cambios. Se repitieron las IT del Service sobre ese esquema. | E02/E03/E04; el intento anterior sin `identity.outbox` se conserva como antecedente fallido. No acredita actualización de la base compartida ni permisos efectivos del usuario real de aplicación. |

**No se añadió DDL en PR79:** sus cuatro archivos son workflow, README, guía y preflight. `identity.outbox`, las columnas de contraseña temporal, índices y permisos ya estaban en la base de comparación. Aplicar los 37 changesets existentes no equivale a haber creado 37 migraciones nuevas para este plan.

#### 3.3. Ajustes en IAM MFE

| ID | Actividad comprobada | Modificación y ubicación | Evidencia y límite |
|---|---|---|---|
| F01 | Corregir el consumidor del contrato de login | R04, `src/adapter/out/auth.ts`: usa `/api/v1/auth/login`, envía `email/password`, lee `TokenResponse` plano con campos snake_case y valida la sesión recibida. | [Adaptador fijado al commit](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/src/adapter/out/auth.ts); `tests/unit/authAdapter.test.ts`; E06/E07. No se modificó el contrato OpenAPI ni el archivo generado para justificar el ajuste. |
| F02 | Ajustar respuestas vacías y manejo de errores | Mismo adaptador: soporta 202/204 sin parsear JSON, conserva código/trace ID/correlación, no reintenta automáticamente un login rechazado; ajustes en hooks de recuperación. | E06 y pruebas de `tests/integration/PasswordResetFlow.test.tsx`. Incluyen dobles de prueba; no todas son llamadas al backend vivo. |
| F03 | Incorporar puerto de autenticación y entrega de sesión al host | R04, `src/application/ports/AuthenticationPort.ts`, `src/application/authentication.tsx`, `src/contracts/remote-props.ts`, `src/App.tsx`, `src/adapter/in/pages/LoginScreen.tsx`: inyección, callback `onAuthenticated` y entrada por correo institucional. | E06; prueba de callback con sesión contractual. Persistencia de tokens, cookies, navegación privada y ciclo del shell no acreditados. |
| F04 | Preparar correlación, transporte del host y proxy local | R04, adaptador y `vite.config.ts`: genera `X-Correlation-Id`, admite fetch inyectado, expone `./authentication`, configura proxy `/api/v1` y `dts.consumeTypes=false`. | E06/E07 y build federado. No fabrica `traceparent`; se requiere contexto real instrumentado. El proxy de desarrollo no acredita la ruta edge de producción. |
| F05 | Añadir CI y requisito de dependencias privadas | R04, `.github/workflows/ci.yml`, `.gitignore`, `.env.example`: Node 24, dependencias compartidas fijadas, requisito `FRONTEND_DEPENDENCIES_READ_TOKEN` y pasos previstos de validación/build. | [CI MFE](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) fallida en el control de credencial; los pasos posteriores aparecen omitidos. Configuración añadida, ejecución completa pendiente. |
| F06 | Documentar integración y ejecutar un smoke HTTP real | R04, `README.md`, `docs/integration/iam-service.md`; drivers locales conservados en E07: adaptador cargado por Vite → proxy → JAR → PostgreSQL, cuenta temporal y limpieza. | Sesión válida, identidad coincidente, correlación y `/auth/me` 200 en E07. No fue un navegador montando el web-shell. El status de login 200 del resumen es literal del driver, no una medición conservada de esa respuesta. |

Los 46 casos del MFE incluyen regresiones existentes de administración y RBAC: no son 46 casos nuevos. La interfaz compartida `IamMfeProps` ya venía de cambios anteriores; esta entrega amplía su integración local con autenticación.

### 4. Pruebas locales, CI y relación con la aceptación

#### 4.1. Resultados conservados y comprobados

| Verificación | Resultado respaldado | Evidencia y precisión |
|---|---|---|
| Service, pruebas unitarias/MVC/contexto | **186 aprobadas**, 0 fallos, 0 errores, 0 omitidas: core 92, adapters 90, API 4. | E01 y suma directa de 38 XML Surefire. Log finaliza `2026-10-20T22:03:22-05:00`. |
| Service, empaquetado JAR | `BUILD SUCCESS` con `package -DskipTests`. | E01; log finaliza `2026-10-20T22:04:33-05:00`. No constituye otra ejecución de pruebas. |
| Service contra PostgreSQL | **55 registrados: 54 aprobados y 1 omitido**, 0 fallos/errores. | E02 y 9 XML Failsafe; finaliza `2026-10-20T22:11:22-05:00`. `OutboxRelayPostgresIT` omitido por ausencia de `IAM_BROKER_HOST`. |
| Base limpia | **37 ejecutados, 0 previos**; migración satisfactoria. | E04, fuente `iam-delivery-migration.log`. Esquema de la base DB `a1e01a4...`, sin DDL nuevo en PR79. |
| Validación y segundo update | Sin errores de validación; **0 nuevos y 37 previos** al repetir. | E04. No prueba rollback ni una migración incremental nueva. |
| Preflight positivo y negativo | Base completa aceptada; demo incompleta rechazada por `Missing IAM table: identity.outbox`. | E04. El rechazo es el resultado esperado de la prueba negativa. |
| JAR local, puerto 18080 | `/health`, `/ready` y `/actuator/prometheus`: 200; correlación, contador, buckets y etiqueta presentes. | E05, resumen conservado. Broker no configurado y agente OTel no cargado en este smoke. |
| MFE/Vitest | **46 aprobadas en 9 archivos**, duración 7,78 s. | E06. El log muestra hora `22:37:43`, sin fecha completa; no se deduce la fecha solo del nombre de la base o del informe. |
| MFE, build federado | Build terminado en 12,89 s; `remoteEntry.js` y tipos federados generados. | E06. `npm run build` incluye `tsc --noEmit && vite build`. |
| MFE, lint y typecheck separados | Se conservan los comandos y logs sin diagnósticos; no el código de salida explícito de cada ejecución separada. | E06. El build sí respalda su paso de TypeScript; no se afirma un resultado local independiente de `check:tokens`. |
| Smoke HTTP MFE/API/DB | Sesión de login válida, identidad coincidente, correlación preservada y `/auth/me` 200 medido. | E07. El driver incluye aserción SQL `1\|1\|1`, pero no hay stdout histórico separado por tabla. |
| Docker | Intento fallido al resolver el dominio de descarga de `eclipse-temurin:21-jre`. | E08. No respalda una imagen nueva construida o desplegada. |

La revisión actual volvió a sumar **47 XML**: 38 Surefire y 9 Failsafe, sin fallos ni errores y con una omisión. No es una repetición de las pruebas. Son 240 casos backend aprobados entre ambas suites y 46 MFE; los agregados no miden cobertura porcentual ni aceptación integral.

El entorno descrito en las guías es Java Temurin 21, Maven 3.9.10, PostgreSQL 16 y Liquibase 4.29; MFE usa Node 24. Los comandos originales dependen de cachés/rutas del equipo. La base local fue aislada; las IT usaron su propietario, no las credenciales reales del usuario de aplicación.

#### 4.2. CI consultada nuevamente en GitHub

| Repositorio y ejecución | Estado remoto comprobado | Interpretación del alcance |
|---|---|---|
| [Service · 39537700071](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071) | `completed / success` para `7c837e0...`. | Workflow `verify` sin DB/broker externos. La evidencia documental conservada registra 186 aprobadas y 55 IT omitidas en CI; las 54 aprobadas contra PostgreSQL son locales. |
| [DB · 39538261790](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) | `completed / success` para `7fe2798...`. | Migración, repetición sin incremento y preflight automatizados con PostgreSQL efímero. |
| [MFE · 39540502857](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) | `completed / failure` para `e73e844...`. | Falló `Verify private dependency access is configured`. Checkouts privados, instalación, lint, typecheck, tests, tokens y build quedaron omitidos. El workflow comprueba que la credencial no esté vacía; este resultado no demuestra permisos válidos ni una nueva ejecución aprobada. |

#### 4.3. Qué criterios de la HU pueden respaldarse

La [HU-INT-IAM-001 / docs#290](https://github.com/code-sena/design-software-docs/issues/290) fue accesible: estaba **abierta**, con etiqueta `estado:refinamiento` y asignación a `nataliachala1`. Sirve como referencia de criterios, no como acta individual del plan.

| Criterio | Evidencia revisada | Conclusión limitada al trabajo disponible |
|---|---|---|
| Base limpia y backend compatible | D01–D04, E02/E04/E05. | Migración y compatibilidad local verificadas; sin nuevos objetos DDL en esta entrega. Falta validar app-user/entorno real. |
| Login con auditoría, refresh y estado del usuario | `LoginPostgresIT`, casos aprobados en E03: `loginSucceeds`, `successResetsTheCounter`, `loginWorksAfterTheLockElapsed`. | Regresión con PostgreSQL real: auditoría SUCCESS, sesión con hash distinto del token, estado de login/contador/bloqueo. Son capacidades existentes verificadas, no desarrollos nuevos atribuidos al plan. |
| Usuario y evento en la misma transacción | `UserManagementPostgresIT#createUserReturnsTemporaryPasswordAndStagedEvent` aprobado; frontera `@Transactional` y llamada a persistencia/outbox en código. | Creación y evento comprobados en el caso. **No se localizó prueba de fallo que demuestre rollback conjunto usuario/outbox**. |
| Reporte de auditoría con autorización | `LoginAuditReportPostgresIT`: `paginatesByCursor`, `filtersByOutcome`, `refusesMissingToken`, `refusesCallerWithoutGrant`; cuatro casos aprobados en el XML original de E03. | Sí hay evidencia de reporte autorizado, filtros/paginación y rechazos 401/403 **mediante MockMvc con PostgreSQL real**. No prueba su consulta desde navegador/shell ni con un usuario del entorno real. |
| JWT validado por otros microservicios, centro/scopes y rotación | No se aportó ejecución específica de un consumidor externo. | No confirmado; el login y `/auth/me` en IAM no satisfacen este bloque. |
| Navegador → shell → MFE → API → persistencia → broker/consumidor | E06/E07 cubren adaptador/proxy/API/DB. | Avance parcial; no montaje del shell en navegador, ni publicación/efecto del evento en consumidor real. |
| Dependencias y trazas | S01–S03 y E05. | Health/readiness/métricas locales comprobados. Sin evidencia IAM de spans exportados, scrape, Loki, dashboard o ciclo de alerta. |

**Corrección documental detectada:** el informe previo y algunas tablas locales agrupan “reporte autorizado” entre lo no acreditado. Deben precisar que sí se probaron cuatro escenarios del reporte en integración local. Lo pendiente es la demostración desde el entorno integrado y su usuario autorizado. Esta distinción evita tanto omitir trabajo verificable como ampliar sus resultados indebidamente.

### 5. Inventario y calidad de las evidencias

La carpeta base de anexos usada en esta revisión es:

`C:/Users/PC/Desktop/design-software-iam-service/design-software-docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/`

Puede abrirse desde el [índice local de evidencias][anexos]. Los originales se localizan bajo R07, según las rutas relativas del manifiesto. Este documento enlaza esas fuentes existentes; no incorpora una copia de los anexos y requiere conservarlas para revisión fuera de este equipo.

| ID | Archivos disponibles | Qué respaldan y qué no |
|---|---|---|
| E01 | `service-tests.txt`; originales `iam-integration-test.log`, `iam-integration-package.log`; XML bajo `R02/iam-core`, `iam-adapters`, `iam-api` en `target/surefire-reports/`. | Resúmenes de 186 aprobadas y empaquetado. Son ejecuciones históricas, con fecha de finalización en los logs Maven. |
| E02 | `postgres-tests.txt`; original `iam-delivery-postgres.log`. | Resultado final de IT, distinto del intento anterior `iam-integration-postgres.log`, fallido contra una demo incompleta. |
| E03 | `service-test-results.json`; originales `R02/iam-api/target/failsafe-reports/TEST-*.xml` y XML Surefire de E01. | Resultado transformado por suite/caso y 47 XML originales accesibles. El recuento se contrastó directamente con los XML. |
| E04 | `db-validation.txt`; originales `iam-delivery-migration.log`, `iam-db-validate.log`, `iam-db-incremental.log`, `iam-db-preflight.log`, `iam-db-preflight-negative.log`. | Creación limpia, repetición sin changesets nuevos y preflight positivo/negativo. No prueba reversión de migraciones. |
| E05 | `runtime-smoke-summary.json`; original `iam-delivery-runtime.log`. | Resumen local del JAR, correlación y métricas. No sustituye captura HTTP completa o evidencia de telemetría externa. |
| E06 | `mfe-tests-build.txt`; originales `iam-mfe-tests.log`, `iam-mfe-build.log`, `iam-mfe-typecheck.log`, `iam-mfe-lint.log`, `iam-mfe-install.log`. | Suite y build local; límites de exit code y fecha indicados en sección 4. |
| E07 | `http-smoke-summary.json`, `mfe-fixture-cleanup.txt`, `smoke-http-driver.ps1.txt`, `smoke-http-adapter.mjs.txt`, `smoke-fixture.java.txt`. | Resumen y procedimiento del smoke real; los drivers son código conservado, no sustituyen resultados históricos ausentes ni vienen preparados para cualquier equipo. |
| E08 | `docker-build-failure.txt`; original `iam-integration-docker-build.log`. | Incidencia de resolución DNS, sin build Docker satisfactorio. |
| E09 | `remote-status.json`, `manifest.json`; consulta actual de PR43/PR79/PR10, runs y HU290. | Instantánea de publicación y CI. Los estados remotos se volvieron a consultar para este informe; la instantánea anterior mantiene su fecha. |

**Integridad comprobada:** los **13 anexos** registrados en `manifest.json` existen y coinciden con sus SHA-256. También existen y coinciden los **23 originales L01–L23** inventariados en ese manifiesto. Su SHA-256 observado es `3c6e32ad23c7723e716138f98e4fa6c0304065d86bf77a08aaa15040c3b225a3`. Un hash comprueba identidad de contenido, no autoría, aprobación ni fecha de ejecución.

El [análisis anterior][analisis-previo] y su inventario JSON son fuentes secundarias: se usaron para localizar evidencia y se contrastaron; no se asumen vigentes todas sus frases. En particular, “ruta propuesta/no existe todavía” pertenece a su primer corte: el expediente sí existe ahora en R05/R06, con cambios locales.

### 6. Documentación que ya se actualizó o preparó

#### 6.1. Documentación incluida en las ramas de implementación

| Repositorio | Archivos | Estado comprobado |
|---|---|---|
| R02 / Service | `README.md`, `docs/observability/ADR-009-r3-observability.md`, `docs/observability/integration.md`, `docs/observability/validation.md`. | Versionados en el commit de PR43. Describen correlación, métricas, configuración y resultados/límites locales. |
| R03 / DB | `README.md`, `docs/integration/iam-service.md`. | Versionados en el commit de PR79. Documentan compatibilidad y comprobación; no nuevas migraciones. |
| R04 / MFE | `README.md`, `docs/integration/iam-service.md`. | Versionados en PR10. Documentan autenticación, puertos, proxy, smoke y dependencias privadas; requieren las precisiones de sección 7. |

“Versionado en una rama” se distingue aquí de “fusionado en `develop`”: las tres PR siguen abiertas.

#### 6.2. Alineación local del macroproyecto: 35 Markdown existentes modificados

En **R05**, `git diff HEAD` registra **35 Markdown existentes modificados**, además del expediente nuevo y anexos. La siguiente tabla enumera esos 35; sus rutas son relativas a R05. Este ajuste existe en esa copia local y no estaba incorporado en R01 al comenzar esta revisión.

| Área | Rutas documentales modificadas | Contenido ajustado en la copia local |
|---|---|---|
| Requisitos · 3 | `04-requirements/non-functional.md`; `04-requirements/traceability-matrix.md`; `04-requirements/user-stories.md`. | Evidencia y estado parcial de la integración IAM; separación de resultados y aceptación pendiente. |
| Arquitectura · 1 | `05-architecture/overview.md`. | Correlación y observabilidad con límites de demostración. |
| Datos · 1 | `06-data/migration-strategy.md`. | Migración limpia, compatibilidad y preflight, sin atribuir DDL nuevo o rollback probado. |
| API · 2 | `07-api/README.md`; `07-api/authentication.md`. | Contrato de login consumido, respuesta plana y responsabilidad del host. |
| UML · 2 | `08-uml/diagram-index.md`; `08-uml/diagrams/source/login-jwt.md`. | Flujo, evidencias y distinción de recorrido probado/pendiente. |
| Servicio IAM · 6 | `09-microservices/services/01-iam-service/README.md`; `components/iam-api/README.md`; `components/iam-api/contract.md`; `data-model.md`; `decisions.md`; `runbook.md` dentro de esa carpeta IAM. | Correlación, métricas, configuración, compatibilidad de datos y referencia a decisión local. |
| DevOps · 5 | `10-devops/README.md`; `10-devops/ci-cd.md`; `10-devops/environments.md`; `10-devops/local-setup.md`; `10-devops/runtime-configuration-map.md`. | CI por repositorio, variables, dependencias y entorno local aislado; límites de imagen/promoción. |
| Calidad e índices · 4 | `11-quality/README.md`; `11-quality/code-review.md`; `11-quality/testing-strategy.md`; `11-quality/plan-mejoramiento/README.md`. | Expediente IAM, cifras locales/CI, omisiones y revisión/DoD pendientes. |
| UX y frontend · 5 | `12-ux-ui/README.md`; `12-ux-ui/navigation-map.md`; `12-ux-ui/mockup-ddd/micro-frontends.md`; `12-ux-ui/mockup-ddd/ux-flows.md`; `12-ux-ui/mockup-ddd/flows/01-auth.md`. | Email, puertos, entrega de sesión y límites del smoke frente al shell. |
| Operación · 2 | `13-operations/README.md`; `13-operations/observability.md`. | Capacidades locales demostradas y observabilidad externa pendiente. |
| Control · 3 | `15-project-control/dependencies.md`; `15-project-control/risks.md`; `15-project-control/technical-backlog.md`. | Acceso CI privado, usuario real, Docker e integración/aceptación por completar. |
| Registro · 1 | `CHANGELOG.md`. | Registro de la alineación documental preparada. No prueba publicación oficial. |

Adicionalmente, R05 incorpora bajo `11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/`:

- `README.md`, `informe-tecnico.md`, `matriz-evidencias.md`, `evidencias/README.md`: **cuatro Markdown nuevos** del expediente.
- Los **13 anexos** de la sección 5 y su `evidencias/manifest.json`.
- `.gitattributes` en la raíz, para preservar el formato textual de los anexos.

El conjunto es de **54 archivos**: 35 Markdown adaptados + 4 Markdown nuevos + 13 anexos + 1 manifiesto + 1 `.gitattributes`. En R05 se observan altas preparadas en el índice y modificaciones locales; eso no acredita commit, push ni aprobación de esta alineación.

#### 6.3. Copia personal bajo `docs/`

R06 contiene la selección anterior bajo `docs/`, junto con `docs/README.md`, [docs/alineacion-documental.md][alineacion-personal] y una adaptación de su README raíz. Explica el origen oficial y la selección personal, y conserva las autorías originales.

Su estado Git observado es `README.md` modificado y `docs/` sin seguimiento, con HEAD `0f78759`. Por tanto, **hay preparación documental local verificable**, pero ese estado no demuestra publicación de `docs/` en el remoto personal. Que R01 tenga el remoto `iam` tampoco prueba que toda esa selección ya esté contenida allí.

#### 6.4. Trabajo documental efectuado en esta revisión

En R01 se creó este informe y se añadió su enlace al índice `11-quality/plan-mejoramiento/README.md`. No se trasladaron automáticamente los 54 archivos de R05/R06, ni se alteraron sus trabajos previos. Este documento queda en revisión y no cambia el estado de las PR o la HU.

### 7. Documentación pendiente de actualizar o reconciliar

| Destino real | Actualización necesaria | Evidencia que debe acompañarla |
|---|---|---|
| R01, documentos enumerados en 6.2 | Revisar e incorporar la alineación IAM preparada en R05 cuando corresponda. Añadir expediente/anexos y enlaces manteniendo autorías y procedencia. La presencia de este informe no significa que todas esas páginas ya estén actualizadas en R01 o en el remoto oficial. | Diff de R05, versiones de sección 2, E01–E09 y verificación de enlaces. |
| R05/R06, `informe-tecnico.md`, `matriz-evidencias.md`, trazabilidad y tablas que afirman “reporte autorizado no acreditado” | Reconocer los cuatro casos aprobados de `LoginAuditReportPostgresIT`; precisar su ejecución con MockMvc/PostgreSQL y mantener pendiente navegador/entorno real. | Código de la suite y XML aprobado de E03. |
| R04, `docs/integration/iam-service.md` y resúmenes que repiten HTTP200 de login | Diferenciar sesión válida y `/auth/me`200 medido del literal `login:200`. Precisar que SQL `1\|1\|1` es aserción del driver sin stdout histórico independiente por tabla. | E07. Para sostener el status exacto del smoke histórico faltaría una captura de la respuesta, o una nueva ejecución fechada expresamente como nueva. |
| R04, guía/README sobre CI, lint y tokens | Actualizar el último diagnóstico: run falla en control de acceso configurado; los pasos siguientes no se ejecutan. Distinguir build/TypeScript comprobados de lint separado sin exit code y `check:tokens` local no acreditado. | E06 y job del run `39540502857`. Un workflow escrito no es un resultado aprobado. |
| R04, `src/contracts/README.md` | Corregir el texto que aún llama placeholder a `remote-props.ts`: ya extiende la interfaz compartida. Conservar procedencia previa y describir las extensiones locales de autenticación. | `src/contracts/remote-props.ts` en el commit revisado. |
| R02, `docs/observability/validation.md` | Conservar el límite “no se ejecutó frontend/shell” como alcance de esa corrida del Service y enlazar separadamente el smoke HTTP posterior del MFE. Incorporar enlaces a anexos accesibles para revisión. | E05/E07 y sus distintos procesos/alcances; no convertirlos en recorrido de navegador. |
| R03, `README.md` y `docs/integration/iam-service.md` | Enlazar validación Service/MFE fijada a SHA y anexos; distinguir repetición sin changesets nuevos de pruebas de rollback, cuatro ambientes o migración incremental nueva. | Diff DB y E04. No atribuir resultados históricos ajenos a PR79. |
| R08/R09, `docs/observability/` | Marcar reportes, checklist, descripción de PR y cierres anteriores como antecedentes cuando conserven estados superados; enlazar PR43 y el corte nuevo. Preservar cambios locales. | Estado Git de ambas copias y comparación con R02. La demo antigua de puerto8080 no acredita el JAR final de puerto18080. |
| Macroproyecto, `07-api/contracts/openapi/iam.yaml`, modelo/diccionario y ADR | No inventar cambios de payload o DDL: el plan corrige consumidor y comprueba esquema existente. Revisar formalización de cabeceras si se acuerda. Mantener `ADR-009-r3-observability.md` como identificador local Service: el ADR009 central corresponde a estrategia frontend. | Contrato/modelo previos y diffs reales. Cualquier cambio del estándar global requiere su decisión correspondiente. |
| `15-project-control/`, HU290 y registros individuales | Registrar pendientes por criterio, responsables acordados y evidencia de aceptación; mantener separados publicación, revisión, fusión y despliegue. Completar acta/bitácora individual cuando estén disponibles. | HU/PR actuales, registro de revisión y resultados que falten. No cerrar por agregado de tests. |

### 8. Limitaciones y evidencias faltantes

Los repositorios de implementación solicitados pudieron revisarse mediante clones locales y el conector GitHub. Estas limitaciones se mantienen explícitas:

| Aspecto | Qué no puede afirmarse | Evidencia necesaria para confirmarlo |
|---|---|---|
| Incorporación oficial del expediente IAM | La consulta del archivo `11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md` en la rama predeterminada de `code-sena/design-software-docs` devolvió **404**. No se confirmó allí la publicación del expediente. Esa respuesta no demuestra su inexistencia en otras ramas ni distingue todas las causas de acceso/ruta. | Commit y ruta accesibles, PR documental y estado de incorporación verificable. |
| Plan individual y reparto de trabajo | No se certifican todas las obligaciones ni el ejecutor de cada prueba. | Acta original y bitácora individual con actividad, fecha y evidencia. |
| Fusión y aceptación | PR43/79/10 y HU290 no están cerradas como entrega aceptada en la consulta. | Revisión, fusión y aceptación registradas. |
| Usuario real de aplicación y base compartida | Las pruebas con propietario de base aislada no prueban conexión/permisos de la cuenta del entorno real, ni actualización compartida. | Ejecución con cuenta de aplicación y entorno identificados, sin divulgar sus credenciales. |
| Atomicidad ante fallo | La transacción implementada y el caso satisfactorio no demuestran rollback conjunto. | Test que fuerce fallo y verifique ausencia de usuario y outbox persistidos. |
| MFE CI | No hay validación remota completa del commit revisado. | Acceso de lectura a dependencias privadas y nueva ejecución completa aprobada. |
| Shell y edge | El smoke no cargó `iamMfe/App` en el web-shell ni validó el ciclo del host. | Navegador con shell real, rutas del entorno, sesión/operación autorizada y evidencia de versiones. |
| Broker, consumidores y JWT externo | No se acredita publicación/consumo ni validación JWT en otros microservicios. | Ejecuciones de esos componentes, eventos/efectos observados y casos JWT/permisos/scopes/rotación. |
| Observabilidad distribuida IAM | Correlación y métricas locales no demuestran Tempo/Loki/scrape, dashboard o alerta con recuperación. | Traza IAM HTTP/JDBC consultable, muestras/scrape, logs y ciclo de alerta identificados por fecha/versión. |
| Imagen y promoción | El fallo Docker no acredita imagen construida ni despliegue. | Build satisfactorio, referencia de imagen y comprobaciones del ambiente desplegado. |
| Tablero y registro HU-log | No se consultó el tablero de Projects ni el repositorio HU-log en esta revisión. El análisis previo registra un 404 de una ruta HU-log, no prueba ausencia del registro. | Acceso verificable a esas fuentes si se necesita certificar su estado. |
| Evidencia de otros equipos | Las capturas y cifras del expediente de Monitoring no prueban IAM. | Evidencia específica de IAM; conservar la entrega de Monitoring con su autoría independiente. |

El informe puede declarar como realizadas las actividades S01–S05, D01–D04 y F01–F06 con los alcances indicados, las ejecuciones históricas respaldadas y la preparación documental localizada. Las acciones de esta sección y de la sección 7 permanecen **pendientes o no confirmadas**, sin incorporarlas como tareas completadas.

## Referencias

- [PR43 · IAM Service](https://github.com/code-sena/design-software-iam-service/pull/43), [PR79 · IAM DB](https://github.com/code-sena/design-software-iam-db/pull/79), [PR10 · IAM MFE](https://github.com/code-sena/design-software-iam-mfe/pull/10).
- [HU-INT-IAM-001 / docs#290](https://github.com/code-sena/design-software-docs/issues/290).
- [Validación Service fijada al commit](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/docs/observability/validation.md).
- [Integración DB fijada al commit](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/docs/integration/iam-service.md).
- [Integración MFE fijada al commit](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/docs/integration/iam-service.md).
- [Índice local del expediente IAM previo][expediente], [anexos y procedencia][anexos], [alineación personal][alineacion-personal], [análisis anterior][analisis-previo].
- [Reglas documentales](../../00-governance/documentation-rules.md), [Definition of Done](../../00-governance/definition-of-done.md).
- [Expediente independiente de Monitoring](r3-observabilidad/README.md).

[expediente]: C:/Users/PC/Desktop/design-software-iam-service/design-software-docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md
[anexos]: C:/Users/PC/Desktop/design-software-iam-service/design-software-docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md
[alineacion-personal]: C:/Users/PC/Desktop/design-software-iam-service/docs-r3-iam/docs/alineacion-documental.md
[analisis-previo]: C:/Users/PC/Desktop/design-software-iam-service/informe-documental-plan-mejoramiento-iam-2026-10-21.md
