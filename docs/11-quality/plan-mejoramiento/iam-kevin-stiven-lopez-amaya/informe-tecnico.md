# Informe técnico de actividades realizadas: IAM

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Cuenta publicadora de las PR de implementación: Kevin-Stiven-Lopez-Amaya404 | Equipo: IAM

## Contexto

Se documentan mejoras de observabilidad e integración realizadas en IAM Service, IAM DB e IAM MFE, junto con resultados que existen en archivos o servicios consultados. Es un informe de trabajo realizado; los aspectos sin evidencia se identifican como límites. No constituye acta de aprobación del plan ni cierre de la integración transversal.

El destino de esta entrega es el repositorio personal `Kevin-Stiven-Lopez-Amaya404/docs-r3-iam`.
La [alineación documental](../../../alineacion-documental.md) registra las copias adaptadas bajo
`docs/`, procedentes de `code-sena/design-software-docs@917d7f6b514a4ff48d86f7ae78273f06e0df8a1c`,
y los archivos nuevos del expediente. Estos cambios documentales corresponden a las copias de
este repositorio; no actualizan la fuente oficial ni acreditan fusión de las PR de código.

La cuenta `Kevin-Stiven-Lopez-Amaya404` publicó las tres PR. No se localizó un acta firmada que enumere todas las obligaciones del plan ni una bitácora que identifique al ejecutor de cada prueba. Debe completarse esa atribución con registros de los participantes; no deducirla de una cuenta de publicación. El expediente Monitoring corresponde a otro autor y dominio.

### Versiones revisadas

| Proyecto | Base | Entrega |
|---|---|---|
| Service | `6987e0fa64628b5602969572d34a900f022ae891` | `r3-observability-delivery-dev` / `7c837e0824263cef0aef89f807731adca839f25e` |
| DB | `a1e01a42692da77269568f0ef49b2e4d3d035fa3` | `r3-iam-db-integration-dev` / `7fe2798cf5d5c90b6e51509e6dd052f1929f81b8` |
| MFE | `8e94e1bd281ca12010fac754e7c376f2d078c20f` | `r3-iam-mfe-integration-dev` / `e73e844bebebb64b99bfce24a908926f0e4cf76b` |

Dependencias MFE: frontend-platform `be8bd79fa0e5e33955c481d928c3867077ff851e` y design-system `b1c8b763b6f812173a11b232e35fed15b4c2f62b`, como repositorios hermanos y dependencias `file:`. La DB usada por las pruebas locales corresponde al esquema base `a1e01a4...`; la entrega DB conserva ese DDL.

Entorno documentado: Java Temurin 21.0.12.1, Maven 3.9.10, PostgreSQL 16, Liquibase 4.29, Node 24.20.0 y npm 11.19.0. Service/DB sitúan sus pruebas el 20 de octubre, America/Bogota; la guía MFE está preparada para el 21. Algunos logs no conservan fecha completa o exit code. No inferir fechas por nombres de archivos/contenedores.

## Contenido

### Actividades y archivos modificados

| ID | Trabajo realizado | Evidencia de implementación |
|---|---|---|
| S01 | Validar/reutilizar `X-Correlation-Id`, generar UUID cuando corresponde, devolverlo y restaurar MDC ante éxito/error. | `CorrelationIdFilter.java`, seis casos `CorrelationIdFilterTest`, dos MVC en `HealthControllerTest`. |
| S02 | Priorizar trace ID de MDC en errores, con alternativas `X-Request-Id` y UUID. | `ApiErrors.java`, tres casos nuevos `ApiErrorsTest`. |
| S03 | Agregar registro Prometheus, exposición operacional, etiqueta `service` e histograma HTTP. | `iam-api/pom.xml`, `application.yml`, un caso nuevo en `IamApiApplicationContextTest`. |
| S04 | Fijar Maven 3.9.10 y crear CI Java 21/verify con reportes XML; excluir archivos locales del contexto Docker. | `.mvn/wrapper/maven-wrapper.properties`, `.github/workflows/ci.yml`, `.dockerignore`. |
| S05 | Documentar decisión, integración y validación de Service. | README y `docs/observability/{ADR-009-r3-observability,integration,validation}.md`. |
| D01 | Crear preflight de solo lectura: grupos NOLOGIN, permisos, cinco esquemas, once tablas, columnas/índices/seeds requeridos. | `scripts/verify-iam-integration.sql`, con `BEGIN READ ONLY`/`ROLLBACK`. |
| D02 | Automatizar DB limpia, validate/update, segundo update sin incremento y preflight. | `.github/workflows/ci.yml`, PostgreSQL 16 efímero y Liquibase 4.29. |
| D03 | Documentar compatibilidad y procedimiento DB/Service. | README y `docs/integration/iam-service.md`. |
| D04 | Aplicar changelog oficial a base aislada y repetir integración fallida contra una demo incompleta. | Logs de migración/validate/incremental/preflight y repetición PostgreSQL; no migraciones nuevas. |
| F01 | Corregir login a email/password, prefijo `/api/v1`, TokenResponse plano snake_case y validación de sesión; un intento por envío. | `src/adapter/out/auth.ts`, `tests/unit/authAdapter.test.ts`. |
| F02 | Aceptar respuestas vacías 202/204 en recuperación y conservar código/trace/correlación del error. | Adaptador, hooks `useRequestReset`/`useConfirmReset`, `PasswordResetFlow.test.tsx`. |
| F03 | Añadir AuthenticationPort/contexto/inyección y esperar `onAuthenticated(session)`; campo Correo institucional. | `AuthenticationPort.ts`, `authentication.tsx`, `remote-props.ts`, `App.tsx`, `LoginScreen.tsx`, fixture `authSession.ts`. |
| F04 | Correlacionar peticiones, admitir transporte instrumentado, exponer `./authentication` y configurar proxy Vite/tipos federados. | Adaptador, `vite.config.ts` (`dts.consumeTypes=false`, pues no consume remotos), `.env.example`. |
| F05 | Crear CI Node 24 con dependencias fijadas y control de credencial privada; ignorar configuración local. | `.github/workflows/ci.yml`, `.gitignore`; segundo commit configura acceso, sin credencial provisionada. |
| F06 | Documentar contrato, host, variables, versiones y verificación MFE. | README y `docs/integration/iam-service.md`. |
| F07 | Preparar cuenta temporal, ejecutar smoke adaptador/proxy/API/DB y limpiar datos/procesos. | Drivers conservados y logs de fixture/HTTP/limpieza en [anexos](./evidencias/README.md). |

Service contiene 12 casos nuevos dentro de una suite de 186; MFE añade cuatro casos del adaptador y ajusta otros dentro de 46. Los escenarios de negocio y PostgreSQL existentes son regresiones ejecutadas, no implementaciones nuevas del plan. Agente OTel del Dockerfile, logs JSON y POM raíz ya existían; no se atribuye aquí una corrección de forks/Surefire.

La PR DB agrega cuatro archivos/cambios de comprobación/automatización/documentación, **sin DDL nuevo**. Outbox/grants/índices existían desde [5dd269c...](https://github.com/code-sena/design-software-iam-db/commit/5dd269c28e4487ca69f4722480920f5aa81512a2), de Jesús Ariel González Bonilla; columnas temporales e índice de recuperación de [PR74](https://github.com/code-sena/design-software-iam-db/pull/74), de ariel5253, fusionada en agosto. MFE corrigió su consumo del OpenAPI existente; no cambió `generated/iam.ts` ni el payload contractual. La interfaz compartida venía de PR8.

### Pruebas locales revisadas

| ID | Ejecución histórica | Resultado | Anexo |
|---|---|---|---|
| T01 | Maven offline `test`, caché `F:/tools/maven-repository` | Core 92 + adapters 90 + API 4 = **186 aprobadas**, cero fallos/errores/omisiones. | [service-tests.txt](./evidencias/service-tests.txt), [47 XML transformados](./evidencias/service-test-results.json). |
| T02 | `package -DskipTests` | JAR empaquetado; no nueva ejecución de tests. | [service-tests.txt](./evidencias/service-tests.txt). |
| T03 | Liquibase update desde cero | **37 ejecutados, 0 previos**, éxito. | [db-validation.txt](./evidencias/db-validation.txt). |
| T04 | Validate y segundo update | Sin errores; **0 nuevos, 37 previos**, conteo estable. | Mismo anexo DB. No acredita rollback o nuevo changeset incremental. |
| T05 | Preflight positivo y negativo | Esquema completo aceptado; demo anterior rechazada por falta de `identity.outbox`. | Mismo anexo DB. El fallo negativo es esperado. |
| T06 | Maven verify contra PostgreSQL real | **54 aprobadas, una omitida**, cero fallos/errores; OutboxRelayPostgresIT sin `IAM_BROKER_HOST`. | [postgres-tests.txt](./evidencias/postgres-tests.txt), JSON de casos. |
| T07 | JAR nuevo en 18080 contra DB aislada | Health/ready/Prometheus200, correlación conservada y series contador/histograma/service presentes. | [runtime-smoke-summary.json](./evidencias/runtime-smoke-summary.json); resumen, no captura HTTP completa. |
| T08 | Build Docker | **Falló por DNS** descargando `eclipse-temurin:21-jre`; no imagen validada. | [docker-build-failure.txt](./evidencias/docker-build-failure.txt). |
| T09 | npm instalación, typecheck y lint | 228 paquetes; guía/PR declaran tipos/lint aprobados. Logs conservan comandos sin diagnósticos ni exit code explícito. | [mfe-tests-build.txt](./evidencias/mfe-tests-build.txt). No equivale a un acta completa de ejecución. |
| T10 | Vitest/jsdom | **46 aprobadas en nueve archivos**, 7,78s; incluye regresiones anteriores. | Mismo anexo MFE. |
| T11 | Build federado | `remoteEntry.js`, App, authentication y tipos; finaliza en 12,89 s. | Mismo anexo MFE. |
| T12 | Adaptador Vite 15101 → JAR 18080 → PostgreSQL | Sesión válida, identidad/correlación correctas, `/auth/me`200 medido. El `login:200` del resumen es literal y no conserva ese status medido. | [http-smoke-summary.json](./evidencias/http-smoke-summary.json), [drivers y límites](./evidencias/README.md). |

Comandos de servicio registrados en su validación, desde el checkout compatible, con credenciales solo por entorno:

```powershell
mvn '-Dmaven.repo.local=F:/tools/maven-repository' -o test
mvn '-Dmaven.repo.local=F:/tools/maven-repository' -o package -DskipTests
mvn '-Dmaven.repo.local=F:/tools/maven-repository' -o `
  '-Dtest=!**/*' '-Dsurefire.failIfNoSpecifiedTests=false' verify
```

DB aislada: `iam_delivery_20261021`, contenedor `iam-r3-demo-20261020`, puerto 55432; se utilizó su propietario. El backend conservó `ddl-auto: validate`. No se acredita aplicación del esquema a una base compartida ni validación del usuario real de aplicación.

El driver del smoke consulta audit SUCCESS, refresh y outbox y exige `1|1|1`; la guía declara comprobación/limpieza. La cuenta temporal tiene actor y rol INSTRUCTOR; no valida la consulta condicional a Actors para `ficha_id` de LEARNER. Se conservan procedimiento, resumen HTTP y salidas INSERT/DELETE; **no stdout SQL histórico separado por tabla**. Un script con aserción no sustituye la salida de su ejecución. Los drivers tienen rutas del equipo y no fueron ejecutados para preparar este informe.

### CI y publicación

| Caso | Resultado al corte | Alcance |
|---|---|---|
| [C01 Service](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071) | Success, [PR43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta | 186 aprobadas, 55 IT omitidos sin servicios externos; artefacto iam-test-reports disponible al consultar. |
| [C02 DB](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) | Success, [PR79](https://github.com/code-sena/design-software-iam-db/pull/79) abierta | Base efímera, validación/migración/repetición/preflight. |
| [C03 MFE](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) | Failure, [PR10](https://github.com/code-sena/design-software-iam-mfe/pull/10) abierta | Control de secreto privado falla; instalación/lint/tipos/tests/tokens/build omitidos. |

MFE necesita `FRONTEND_DEPENDENCIES_READ_TOKEN` con Contents:read sobre design-system y frontend-platform. El run anterior falló al descargar el repositorio privado; el último explicita el secreto ausente. No se acredita su configuración o repetición verde. La [instantánea remota](./evidencias/remote-status.json) conserva fecha de consulta, cuenta, títulos en inglés, refs y conclusiones; no acredita checks obligatorios o despliegue.

El repositorio de trabajo original `Kevin-Stiven-Lopez-Amaya404/design-software-iam-service`
conservaba el commit inicial `97239a8`; tener clones/logs dentro de su carpeta no los publicaba
en ese remoto. El destino documental actual es `Kevin-Stiven-Lopez-Amaya404/docs-r3-iam`:
los anexos revisados incluidos aquí se publicarán con esta entrega, junto con las copias adaptadas.
La publicación de código que este informe acredita sigue siendo la de las PR oficiales abiertas.
No se incorporan logs ni XML originales completos; se entregan extractos/resultados transformados,
con procedencia, límites y hashes en el manifiesto.

### Incidencias y límites

- La integración inicial falló en la demo sin outbox. Se resolvió usando una base nueva con changelog oficial completo y repitiendo T06, sin crear migraciones de negocio. El fallo previo y el resultado final se distinguen en el manifiesto y validación Service.
- La compilación anterior en Windows tuvo problemas de acceso/resolución; la entrega nueva aprobó T01 sin cambiar el POM ni desactivar forks. Los informes de la copia anterior conservan valor histórico, no describen el resultado final.
- Docker no produjo imagen validada. MFE local satisfactorio no sustituye el CI bloqueado.
- Login, auditoría, refresh no plano y estado de usuario tienen casos PostgreSQL aprobados; creación tiene evento staged. La frontera `@Transactional` se revisó en código, pero **no se localizó prueba de rollback conjunto usuario+outbox ante fallo**.
- No se acredita shell real en navegador, ruta edge desplegada, cookies/refresh/logout del host, JWT verificado por otro microservicio/rotación, broker/consumidores o efecto de negocio externo.
- El smoke JAR no cargó agente OTel. Endpoint Prometheus no demuestra scrape; correlación no demuestra spans. Sin evidencia IAM de Tempo, Loki, dashboard, alerta/disparo/recuperación o pruebas de carga/cobertura porcentual.
- No se acredita revisión/fusión de las PR ni aceptación formal del plan. HU#290 sigue abierta; el estado vivo está en issue/board. Su asignación a nataliachala1 y publicación de PR por Kevin requieren coordinación, no reasignación automática.

### Copias documentales alineadas en este repositorio personal

Esta entrega incorpora el expediente y sus anexos y alinea las copias de [trazabilidad](../../../04-requirements/traceability-matrix.md), [historias](../../../04-requirements/user-stories.md), NFR, arquitectura/correlación, [IAM](../../../09-microservices/services/01-iam-service/README.md), [DevOps](../../../10-devops/README.md), [estrategia de pruebas](../../testing-strategy.md), [operación](../../../13-operations/README.md), [API](../../../07-api/README.md), [UX/MFE](../../../12-ux-ui/README.md) y [control del proyecto](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/15-project-control/README.md) incluidas bajo `docs/`. Los textos distinguen baseline histórico de entrega en rama. La fuente oficial de `code-sena/design-software-docs` permanece separada de estas adaptaciones: este informe no acredita que haya recibido o aprobado los cambios. No se modifican DDL, OpenAPI ni ADR009 central para aparentar avances; el ADR009 de Service se enlaza como decisión local, evitando colisión de numeración. El [índice de alineación](../../../alineacion-documental.md) permite revisar el alcance de las copias.

El registro hu-log y el board no se verificaron: la consulta a `design-software-hu-log/contents/log` devolvió404, que no distingue falta de acceso/ruta/repositorio. No se afirma que falten o estén actualizados. Para cierre, completar responsables, registros accesibles y aceptación según DoD; ningún pendiente se presenta aquí como trabajo realizado.

## Referencias

- [Guía Service fijada a commit](https://github.com/code-sena/design-software-iam-service/blob/7c837e0824263cef0aef89f807731adca839f25e/docs/observability/validation.md).
- [Guía DB fijada a commit](https://github.com/code-sena/design-software-iam-db/blob/7fe2798cf5d5c90b6e51509e6dd052f1929f81b8/docs/integration/iam-service.md).
- [Guía MFE fijada a commit](https://github.com/code-sena/design-software-iam-mfe/blob/e73e844bebebb64b99bfce24a908926f0e4cf76b/docs/integration/iam-service.md).
- [Alineación documental y procedencia de las copias](../../../alineacion-documental.md).
- [Matriz cambio/criterio/prueba/evidencia](./matriz-evidencias.md), [manifest](./evidencias/manifest.json), [DoD](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/00-governance/definition-of-done.md).
