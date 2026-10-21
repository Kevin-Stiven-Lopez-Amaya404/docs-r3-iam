# Evidencias conservadas de la entrega IAM

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Fuentes de implementación/publicación: Kevin-Stiven-Lopez-Amaya404 | Equipo: IAM

## Contexto

Anexos preparados a partir de archivos ya existentes y una consulta de metadatos GitHub. No son una nueva ejecución ni un acta de cierre. Los logs originales viven en el espacio local `design-software-iam-service` y muchos están ignorados por Git; aquí se incorporan extractos/resultados revisados que pueden acompañar la documentación.

El [manifiesto](./manifest.json) identifica fuente, SHA256 original, transformación y hash del archivo preparado. Un hash acredita identidad, no autor/fecha de ejecución. Los resultados XML se reducen a suite/casos/estado: no se publican properties, system-out/system-err ni detalles de error. Estos resultados locales son distintos del artefacto remoto `iam-test-reports` de CI.

Los anexos publicados usan UTF-8 y saltos de línea LF; `.gitattributes` conserva ese formato al descargar en Windows o Unix para mantener verificables sus hashes. Las transcodificaciones y selecciones se registran en el manifiesto, separadas del hash original.

## Contenido

| Archivo | Evidencia y alcance |
|---|---|
| [manifest.json](./manifest.json) | Procedencia y hashes de anexos; versiones y límites. |
| [service-tests.txt](./service-tests.txt) | Resúmenes Maven de 186 aprobadas y empaquetado sin tests. Prefijos L indican línea original. |
| [postgres-tests.txt](./postgres-tests.txt) | Integración final:55 registrados,54 aprobados y1 omitido, BUILD SUCCESS. |
| [service-test-results.json](./service-test-results.json) | Transformación de47 XML locales, con nombres y estados por caso;186+54 aprobados y1 omitido. |
| [db-validation.txt](./db-validation.txt) | Migración37 desde cero, validate, segundo update y preflight positivo/negativo. |
| [runtime-smoke-summary.json](./runtime-smoke-summary.json) | Resumen JAR 18080: health/ready/Prometheus200, correlación y métricas presentes. Sin captura HTTP completa. |
| [mfe-tests-build.txt](./mfe-tests-build.txt) |46 casos/nueve archivos, build y comandos tipos/lint; estos dos últimos sin exit code explícito conservado. |
| [http-smoke-summary.json](./http-smoke-summary.json) | Sesión/identidad/correlación y me200; `login:200` es literal del driver. |
| [mfe-fixture-cleanup.txt](./mfe-fixture-cleanup.txt) | Salidas INSERT/DELETE; por sí solas no identifican consultas ni tablas. |
| [smoke-http-driver.ps1.txt](./smoke-http-driver.ps1.txt) | Código local conservado: fixture, JAR/Vite, SQL1\|1\|1 y limpieza. Copia textual, sin reejecución. |
| [smoke-http-adapter.mjs.txt](./smoke-http-adapter.mjs.txt) | Driver Vite que carga adaptador y aserta sesión/correlación/identidad/me200. |
| [smoke-fixture.java.txt](./smoke-fixture.java.txt) | Generador efímero de hash BCrypt/material RSA usado por el driver, no valores de claves generadas. |
| [docker-build-failure.txt](./docker-build-failure.txt) | Fallo DNS al descargar imagen base; no acredita imagen construida. |
| [remote-status.json](./remote-status.json) | Consulta GitHub fechada: PR, autores, refs, títulos y resultado de runs; no ejecución nueva. |

### Interpretación y reproducción

- La fuente final PostgreSQL es `iam-delivery-postgres.log`. `iam-integration-postgres.log` es el intento fallido anterior contra demo incompleta y no sustituye el resultado final; su hash se conserva como antecedente en el inventario local del análisis. El preflight negativo incluido demuestra el diagnóstico de outbox ausente sin modificar esa demo.
- No se guardó salida SQL histórica separada de los tres conteos del smoke. El procedimiento contiene aserción1\|1\|1; la guía declara éxito, pero no se presenta el código como sustituto de esa salida.
- Los drivers `.txt` conservan rutas locales (`F:/tools/...`, nombres de clones/contenedor). Requieren revisar configuración y dependencias para reproducirse en otro equipo; no están presentados como herramienta portable. No se incluyen contraseñas, JWT o claves privadas generadas.
- `login:200` del resumen HTTP fue impreso como literal; sí se exige login satisfactorio con sesión válida y se comprueba me.status===200. El test PostgreSQL loginSucceeds, por separado, sí aserta loginHTTP200.
- Una integración omitida por falta de broker no se cuenta aprobada. CI Service omite55IT; las54 aprobadas de PostgreSQL corresponden a ejecución local.
- Los resultados no acreditan usuario real de aplicación, base compartida, rollback usuario/outbox ante fallo, shell/navegador, consumidores, scrape, Tempo/Loki/alertas o despliegue.

## Referencias

- [Informe técnico](../informe-tecnico.md), [Matriz](../matriz-evidencias.md), [Índice IAM](../README.md).
- [Service CI](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071), [DB CI](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790), [MFE CI](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857).
