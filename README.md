# Plan de mejoramiento IAM — documentación y evidencias

> Estado: en revisión | Corte documental: 2026-10-21, America/Bogota
> Repositorio personal: Kevin-Stiven-Lopez-Amaya404/docs-r3-iam

Entrega documental del trabajo realizado en **IAM Service, IAM DB e IAM MFE**:
actividades, cambios de código, pruebas locales, resultados de CI y evidencias
conservadas. Los resultados corresponden a ejecuciones anteriores; preparar
esta documentación no implicó ejecutar nuevamente las pruebas.

## Documentos para la entrega

| Documento | Contenido |
|---|---|
| [Expediente IAM](docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) | Alcance, versiones y lectura del conjunto. |
| [Informe técnico](docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md) | Actividades realizadas, archivos modificados, pruebas y límites. |
| [Matriz de evidencias](docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) | Relación cambio → criterio → prueba → resultado → evidencia. |
| [Anexos](docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md) | Extractos de logs, resultados de casos, resúmenes, drivers y procedencia. |
| [Manifiesto de evidencias](docs/11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/manifest.json) | Archivos de origen, transformaciones y hashes SHA-256. |
| [Índice documental](docs/README.md) | Documentos adaptados por área del macroproyecto. |
| [Alineación documental](docs/alineacion-documental.md) | Procedencia, adaptación y alcance de las copias incluidas. |

## Resultados registrados

| Componente | Pruebas y trabajo local | Estado de publicación y CI al corte |
|---|---|---|
| Service | **186 casos aprobados**. Integración PostgreSQL: **54 aprobados y uno omitido** por falta de broker. JAR y smoke de health/ready/Prometheus satisfactorios. | [PR #43](https://github.com/code-sena/design-software-iam-service/pull/43) abierta, sin fusionar. [CI aprobada](https://github.com/code-sena/design-software-iam-service/actions/runs/39537700071): 186 aprobados y **55 IT omitidos** en ese entorno. |
| DB | 37 changesets existentes aplicados desde cero; validación, segundo update sin cambios y preflight positivo/negativo. La entrega no añade DDL. | [PR #79](https://github.com/code-sena/design-software-iam-db/pull/79) abierta, sin fusionar. [CI aprobada](https://github.com/code-sena/design-software-iam-db/actions/runs/39538261790). |
| MFE | **46 casos aprobados**, build federado y smoke HTTP del adaptador mediante proxy hacia API/PostgreSQL. | [PR #10](https://github.com/code-sena/design-software-iam-mfe/pull/10) abierta, sin fusionar. [CI fallida](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857) por ausencia de `FRONTEND_DEPENDENCIES_READ_TOKEN`, antes de instalar o probar. |

Las 54 integraciones PostgreSQL aprobadas son **locales** y se cuentan aparte
de los 186 casos de servicio. El caso omitido no se cuenta como aprobado.
El build Docker falló por DNS antes de compilar; no hay una imagen nueva
validada por ese intento.

El smoke HTTP no montó el web-shell en un navegador. Quedan sin acreditar
el usuario real de aplicación, la base compartida, rollback conjunto
usuario/outbox ante fallo, broker/consumidores y telemetría distribuida IAM
(scrape, Tempo, Loki, dashboard y alerta con recuperación). Los detalles y
la calidad de cada evidencia están en el informe y la matriz.

## Procedencia, autoría y alcance

`docs/` contiene una **selección de copias adaptadas** de la documentación del
macroproyecto, basada en
[`code-sena/design-software-docs` @ `917d7f6b514a4ff48d86f7ae78273f06e0df8a1c`](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c),
y el expediente específico de esta entrega IAM. Se conservan las autorías
de los documentos de origen; no se incluyen el expediente de Monitoring
ni sus capturas o resultados como evidencia IAM.

Las PR de implementación fueron publicadas por `Kevin-Stiven-Lopez-Amaya404`.
Esa cuenta identifica la publicación y los commits revisados, pero no permite
atribuir a una sola persona la ejecución de todas las pruebas. El reparto
entre participantes requiere su bitácora o registro individual.

Esta entrega personal **no actualiza la fuente oficial del macroproyecto**,
no sustituye su tracker y no certifica aceptación o cierre del plan.
La incorporación oficial requiere el proceso de revisión correspondiente;
los estados GitHub aquí resumidos son un corte fechado, no información en vivo.
