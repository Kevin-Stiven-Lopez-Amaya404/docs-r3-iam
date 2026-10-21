# Alineación documental de la entrega IAM

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Procedencia y autoría: documentos originales de code-sena/design-software-docs; autores conservados en cada archivo. Expediente de entregas publicadas por Kevin-Stiven-Lopez-Amaya404 | Equipo: IAM

## Destino y alcance de esta copia

El destino de esta preparación documental es el repositorio personal [Kevin-Stiven-Lopez-Amaya404/docs-r3-iam](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam). Reúne copias adaptadas de documentación del macroproyecto y un expediente IAM con evidencias de trabajo realizado. Permite revisar la entrega personal; **no sustituye la fuente oficial de documentación ni acredita que estos ajustes se hayan incorporado a ella**.

La fuente documental contrastada es [code-sena/design-software-docs, commit 917d7f6b514a4ff48d86f7ae78273f06e0df8a1c](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c), árbol Git `885ce4415042b9ef319ed6e199cbceb890710176`. Se conservan los autores originales de los documentos adaptados. La cuenta que publica este repositorio o las PR de implementación no pasa a ser autora de todo el contenido previo, ni identifica por sí sola al ejecutor de cada prueba.

En el repositorio oficial las secciones numeradas están en la raíz. Aquí se agrupan bajo `docs/` para organizar la entrega personal; ese prefijo es una adaptación de empaquetado, no una carpeta que se atribuya al baseline oficial. Las referencias a material del macroproyecto no incluido en esta selección apuntan al baseline oficial; los documentos incluidos se relacionan mediante enlaces locales.

El expediente IAM es independiente del trabajo de Monitoring / Kevin Andrey Culma Gómez. Sus capturas Tempo, alertas y resultados no se incorporan como evidencia de IAM o de Kevin Stiven. Tampoco se modifica el DDL, el OpenAPI ni el estado del board mediante estas copias.

## Inventario importado y naturaleza de los archivos

La selección de origen comprende **54 archivos**. No son 54 documentos originales nuevos ni 54 cambios de código:

| Grupo importado | Cantidad | Naturaleza |
|---|---:|---|
| Markdown existentes del macroproyecto | 35 | Copias adaptadas para reflejar implementación, resultados, publicación y límites de IAM. Su autoría original se conserva. |
| Markdown del expediente IAM | 4 | Índice de entrega, informe técnico, matriz y guía de anexos. |
| Anexos de evidencia | 13 | Extractos, resultados transformados, resúmenes históricos y drivers conservados; no nuevas ejecuciones. |
| Manifiesto de evidencia | 1 | Procedencia, transformaciones, versiones y hashes. |
| `.gitattributes` | 1 | Conservación del formato textual de los anexos para verificar hashes. |
| **Total importado** | **54** | Selección documental con contenido previo adaptado y expediente nuevo. |

Este índice `docs/alineacion-documental.md` y la adaptación del README del repositorio son elementos de presentación del destino personal, adicionales a esos 54 archivos importados. El conteo no representa el total futuro del repositorio.

## Mapa de los 35 Markdown adaptados

Las rutas de esta tabla son relativas a `docs/`. Describen el propósito de los ajustes y no sustituyen la matriz cambio → prueba → evidencia del expediente.

| Grupo y cantidad | Documentos incluidos | Alineación realizada |
|---|---|---|
| Requisitos — 3 | [non-functional.md](./04-requirements/non-functional.md), [traceability-matrix.md](./04-requirements/traceability-matrix.md), [user-stories.md](./04-requirements/user-stories.md) | Relacionar NFR e integración IAM con entregas en rama y pruebas disponibles; distinguir aceptación pendiente de resultados locales. No cerrar HU por la existencia de una PR. |
| Arquitectura — 1 | [overview.md](./05-architecture/overview.md) | Separar correlación HTTP, métricas y configuración OTel de una traza distribuida efectivamente consultada. |
| Datos — 1 | [migration-strategy.md](./06-data/migration-strategy.md) | Registrar 37 changesets heredados, update repetido y preflight. Aclarar que la mejora DB no incorpora DDL ni acredita rollback o permisos del usuario real. |
| API — 2 | [README.md](./07-api/README.md), [authentication.md](./07-api/authentication.md) | Reflejar el contrato real de login, la corrección de su consumidor MFE y el alcance de los ensayos; no atribuir cambios al OpenAPI existente. |
| UML — 2 | [diagram-index.md](./08-uml/diagram-index.md), [login-jwt.md](./08-uml/diagrams/source/login-jwt.md) | Alinear el flujo login/JWT con persistencia, auditoría, correlación y límites de lo probado. |
| Microservicio IAM — 6 | [README.md](./09-microservices/services/01-iam-service/README.md), [data-model.md](./09-microservices/services/01-iam-service/data-model.md), [decisions.md](./09-microservices/services/01-iam-service/decisions.md), [runbook.md](./09-microservices/services/01-iam-service/runbook.md), [iam-api/README.md](./09-microservices/services/01-iam-service/components/iam-api/README.md), [iam-api/contract.md](./09-microservices/services/01-iam-service/components/iam-api/contract.md) | Precisar componentes, esquema existente, contrato, decisión local de observabilidad, configuración operativa y evidencia de regresión. Distinguir ADR local del Service y numeración central. |
| DevOps — 5 | [README.md](./10-devops/README.md), [ci-cd.md](./10-devops/ci-cd.md), [local-setup.md](./10-devops/local-setup.md), [runtime-configuration-map.md](./10-devops/runtime-configuration-map.md), [environments.md](./10-devops/environments.md) | Sustituir afirmaciones de CI inexistente por resultados por componente; separar entorno aislado, configuración de Compose y despliegue. Documentar variables API/DB/proxy y secreto de lectura de dependencias. |
| Calidad e índices — 4 | [README.md](./11-quality/README.md), [testing-strategy.md](./11-quality/testing-strategy.md), [code-review.md](./11-quality/code-review.md), [plan-mejoramiento/README.md](./11-quality/plan-mejoramiento/README.md) | Enlazar el expediente IAM sin mezclarlo con Monitoring; distinguir suites locales, CI, casos omitidos, revisión y DoD pendientes. |
| UX y MFE — 5 | [README.md](./12-ux-ui/README.md), [navigation-map.md](./12-ux-ui/navigation-map.md), [micro-frontends.md](./12-ux-ui/mockup-ddd/micro-frontends.md), [ux-flows.md](./12-ux-ui/mockup-ddd/ux-flows.md), [01-auth.md](./12-ux-ui/mockup-ddd/flows/01-auth.md) | Actualizar correo institucional, sesión entregada al host, puertos/inyección y proxy; diferenciar smoke HTTP del adaptador y montaje del shell en navegador. |
| Operación — 2 | [README.md](./13-operations/README.md), [observability.md](./13-operations/observability.md) | Registrar health/readiness/Prometheus y correlación comprobados, sin declarar scrape, Tempo, Loki, dashboards o alertas no observados. |
| Control del proyecto — 3 | [dependencies.md](./15-project-control/dependencies.md), [risks.md](./15-project-control/risks.md), [technical-backlog.md](./15-project-control/technical-backlog.md) | Reflejar dependencias privadas, riesgo CI, usuario de aplicación, aceptación E2E y Docker/promoción. Conservar el conteo global de agosto como histórico, sin inventar estado actual de otros servicios. |
| Registro de cambios — 1 | [CHANGELOG.md](./CHANGELOG.md) | Registrar la actualización documental y su alcance para revisión. Su registro en esta copia no demuestra publicación en el repositorio oficial. |
| **Total** | **35 Markdown existentes adaptados** | Los encabezados y atribuciones de origen se mantienen separados de la publicación personal. |

## Expediente y evidencias incluidas

Los cuatro Markdown del expediente son el [índice IAM](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md), el [informe técnico](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md), la [matriz de evidencias](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) y la [guía de anexos](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md). La matriz relaciona actividades S/D/F, criterios de HU, pruebas T y CI C con evidencia y límites.

| Anexos incluidos — 13 archivos | Contenido verificable y límite |
|---|---|
| [service-tests.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/service-tests.txt), [postgres-tests.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/postgres-tests.txt), [service-test-results.json](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/service-test-results.json) | Resúmenes Service y transformación de 47 XML locales con casos/estados. Se excluyen propiedades y salidas completas; no equivalen a ejecución nueva ni a los artefactos remotos de CI. |
| [db-validation.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/db-validation.txt) | Migración limpia, validate, repetición sin cambios y preflight positivo/negativo. El fallo por outbox ausente es la condición negativa esperada. |
| [runtime-smoke-summary.json](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/runtime-smoke-summary.json) | Resumen JAR: health/readiness/Prometheus, correlación y métricas. No captura HTTP completa ni telemetría distribuida. |
| [mfe-tests-build.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/mfe-tests-build.txt) | Vitest y build federado. Lint/typecheck conservan comandos sin diagnósticos ni exit code explícito; no hay resultado local separado de `check:tokens`. |
| [http-smoke-summary.json](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/http-smoke-summary.json), [mfe-fixture-cleanup.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/mfe-fixture-cleanup.txt) | Sesión/identidad/correlación, me200 y salidas fixture/limpieza. `login:200` es literal del driver y no el status histórico medido; no se conserva stdout SQL separado por tabla. |
| [smoke-http-driver.ps1.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/smoke-http-driver.ps1.txt), [smoke-http-adapter.mjs.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/smoke-http-adapter.mjs.txt), [smoke-fixture.java.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/smoke-fixture.java.txt) | Procedimiento conservado con aserciones y rutas del equipo de origen. No se reejecutó al preparar documentación; no sustituye la salida histórica ausente ni es una herramienta portable lista para cualquier entorno. |
| [docker-build-failure.txt](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/docker-build-failure.txt) | Fallo DNS al descargar la imagen base. Acredita incidencia y límite, no imagen construida. |
| [remote-status.json](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/remote-status.json) | Instantánea fechada de PR, cuentas, refs, títulos y CI consultadas. El estado vivo permanece en GitHub. |

El [manifiesto](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/manifest.json) es un archivo adicional a esos 13 anexos: identifica fuentes, versiones, transformaciones y SHA256. [.gitattributes](./.gitattributes) conserva los formatos textuales previstos. Un hash verifica identidad de contenido; no certifica por sí solo la fecha, el ejecutor o la aprobación de una prueba.

## Resultados y pendientes que debe preservar la documentación

| Ámbito | Hecho respaldado al corte | Pendiente o límite exacto |
|---|---|---|
| Service local | 186 aprobadas; PostgreSQL: 54 aprobadas y una omitida. Login/auditoría SUCCESS, refresh no plano y estado del usuario tienen casos registrados. | No sumar el empaquetado con tests omitidos como otra suite. La frontera transaccional está en código; falta prueba de rollback conjunto usuario/outbox ante fallo. |
| Service CI | [Run 39537700071](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071): success, 186 aprobadas y 55 IT omitidos. | No trasladar las 54 aprobadas locales a CI ni contar omitidos como aprobados; faltan dependencias externas para ejecutar IT allí. |
| DB local/CI | 37 changesets heredados desde cero, validate, segundo update con 0 nuevos y preflight. [Run 39538261790](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790) aprobado. | No hay nuevo DDL. ROLLBACK del script de lectura no prueba reversión de migraciones. La prueba usó propietario de base aislada; faltan usuario real de aplicación y entorno compartido. |
| MFE local | 46 casos aprobados y build federado; smoke del adaptador Vite→API→PostgreSQL con sesión válida e identidad/correlación. | No equivale a 46 E2E vivos. Falta shell en navegador, rutas edge del entorno, cookies/refresh/logout del host y stdout SQL histórico separado de los conteos. |
| MFE CI | [Run 39540502857](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857): failure por `FRONTEND_DEPENDENCIES_READ_TOKEN` ausente. | Configurar lectura de frontend-platform/design-system y obtener ejecución completa; instalación/lint/tipos/tests/tokens/build no se ejecutaron en ese run. No publicar el valor del secreto. |
| Imagen y despliegue | Build Docker falló por resolución DNS al descargar `eclipse-temurin:21-jre`. | Falta imagen nueva construida/probada y evidencia de despliegue/promoción; los puertos del JAR directo no acreditan el edge de Compose. |
| Integración y observabilidad | Correlación HTTP y endpoint/series Prometheus comprobados localmente; agente OTel y otras piezas tenían antecedentes previos. | Sin evidencia IAM de broker/consumidor real, verificación JWT downstream/rotación, spans OTLP/Tempo, scrape, Loki, dashboard, alerta/disparo/recuperación o carga/cobertura porcentual. |
| Publicación y aceptación | [Service PR43](https://github.com/code-sena/design-software-iam-service/pull/43), [DB PR79](https://github.com/code-sena/design-software-iam-db/pull/79) y [MFE PR10](https://github.com/code-sena/design-software-iam-mfe/pull/10) abiertas, sin fusionar, en la consulta conservada; autor de publicación Kevin-Stiven-Lopez-Amaya404. | No acredita fusión, promoción, aceptación de HU#290 ni cierre del plan. La publicación de esta copia personal no modifica esos estados. |
| Atribución y registros | Fuentes y autores Git identificables; informe limitado a trabajo disponible. | Falta acta original firmada y reparto individual de actividades/pruebas. Board y hu-log no se verificaron; un 404 de consulta no demuestra ausencia del registro. |

Preparar esta selección y sus anexos fue trabajo documental. No se ejecutaron nuevas pruebas para convertir pendientes en resultados. Las acciones restantes se mantienen diferenciadas del informe de actividades realizadas.

## Referencias de lectura

- [Informe técnico y versiones exactas](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md).
- [Matriz cambio/criterio/prueba/evidencia](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md).
- [Anexos, procedencia y límites](./11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md).
- [Fuente documental oficial fijada al baseline](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c).
- [HU-INT-IAM-001 / docs#290](https://github.com/code-sena/design-software-docs/issues/290).
