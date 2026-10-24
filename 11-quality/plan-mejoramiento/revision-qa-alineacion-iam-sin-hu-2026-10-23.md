# Revisión QA de entrega técnica — alineación IAM (sin HU)

> Este documento sigue como referencia la estructura de `Plantilla_qa.md`, pero el objeto revisado es una entrega técnica de alineación entre ramas. **No corresponde a una Historia de Usuario** y no se asignan criterios, bugs ni dependencias de HU.

## Actualización del estado vigente — 24/10/2026

Esta sección reemplaza los estados de rama/CI que aparecen en el corte histórico del 23 de octubre más abajo. La actualización documental y los metadatos consultados en GitHub reflejan el siguiente estado:

| Repositorio y rama de Kevin | `develop` comparado | Resultado remoto | PR |
|---|---|---|---|
| [IAM Service — `r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev) | Base `c7735cfc3135aad83091e2cfac5abe5809853c85`; punta `06ab67bd54c0a561e20dde784c0191d67df30e6c` | 4 commits adelante, **0 detrás**. CI IAM y negativos R4 aprobaron en la punta actual. | [#43 abierta](https://github.com/code-sena/design-software-iam-service/pull/43) |
| [IAM DB — `r3-iam-db-integration-dev`](https://github.com/code-sena/design-software-iam-db/tree/r3-iam-db-integration-dev) | Base `a1e01a42692da77269568f0ef49b2e4d3d035fa3`; punta `b89574a7eac05e2edef4d8d3f61df08f40214ce7` | 2 commits adelante, **0 detrás**. La CI migró una base limpia, comprobó actualización incremental y ejecutó el preflight. | [#79 abierta](https://github.com/code-sena/design-software-iam-db/pull/79) |
| [IAM MFE — `r3-iam-mfe-integration-dev`](https://github.com/code-sena/design-software-iam-mfe/tree/r3-iam-mfe-integration-dev) | Base `8e94e1bd281ca12010fac754e7c376f2d078c20f`; punta `301cf6c4f5d07ddf2677bf3d4198ea8481e2720e` | 3 commits adelante, **0 detrás**. La CI falló antes de instalar dependencias por falta de `FRONTEND_DEPENDENCIES_READ_TOKEN`; build y pruebas CI quedaron omitidos. | [#10 abierta](https://github.com/code-sena/design-software-iam-mfe/pull/10) |

Los tres PR siguen abiertos y apuntan a `develop`; ninguno se cerró ni se reemplazó. Los commits documentales que fijan la procedencia actual en cada rama aparecen con autor y committer `Kevin-Stiven-Lopez-Amaya404`: [Service](https://github.com/code-sena/design-software-iam-service/commit/06ab67bd54c0a561e20dde784c0191d67df30e6c), [DB](https://github.com/code-sena/design-software-iam-db/commit/b89574a7eac05e2edef4d8d3f61df08f40214ce7) y [MFE](https://github.com/code-sena/design-software-iam-mfe/commit/301cf6c4f5d07ddf2677bf3d4198ea8481e2720e).

**Dictamen vigente:** la alineación de historial con `develop` está comprobada en los tres repositorios; eso no certifica por sí solo integración funcional de los tres candidatos. El check MFE continúa bloqueado hasta que un administrador configure el secreto indicado, y se debe repetir el smoke con los SHA actuales de Service/DB/MFE. En Service permanece la diferencia entre el permiso `VIEW` del OpenAPI fijado y `MANAGE` del código al revocar sesiones ajenas; la fuente canónica de contratos debe resolverla antes de regenerar la copia/tipos. Las pruebas DB/broker omitidas, shell real y observabilidad distribuida tampoco se declaran aprobadas. Por ello, **no se afirma “sin inconvenientes” ni aprobación final QA**.

Los resultados enlazados en el cuerpo de este informe con corte 23/10/2026 son históricos, aunque fueron válidos para esos SHA. Las descripciones actuales de los PR se actualizaron para distinguir evidencia histórica, checks de las puntas vigentes y pendientes.

## 1. Información general

| Campo | Información |
|---|---|
| **Objeto de revisión** | Alineación de `r3-observability-delivery-dev` con `develop` en IAM Service |
| **Tipo de entrega** | Integración de cambios de referencia, preservación de observabilidad R3 y documentación de validación |
| **Identidad Git de los commits de esta entrega** | `Kevin-Stiven-Lopez-Amaya404` (`mrroobot08@gmail.com`), confirmada por los metadatos de GitHub |
| **Responsable QA** | No indicado en la evidencia disponible |
| **Fecha de corte original** | 23/10/2026, America/Bogota; estado vigente actualizado en la sección anterior |
| **Ambiente revisado** | GitHub (rama, PR y GitHub Actions) y ejecución Maven local documentada |
| **Asignación individual** | Kevin Stiven López Amaya: R3 de observabilidad en IAM Service, según el Acta 02 y el Anexo de Aporte Individual revisados |
| **PR** | [#43 — feat(iam): integrate request correlation and Prometheus metrics](https://github.com/code-sena/design-software-iam-service/pull/43) |
| **Repositorio de trabajo asignado** | [`code-sena/design-software-iam-service`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev), rama `r3-observability-delivery-dev` |
| **Rama de referencia** | [`develop`](https://github.com/code-sena/design-software-iam-service/tree/develop), SHA `c7735cfc3135aad83091e2cfac5abe5809853c85` |
| **Rama revisada en el corte histórico** | [`r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev), SHA `5350e1495c7764a62cd47bc9e83063a7eeb68053`; la punta vigente es `06ab67bd54c0a561e20dde784c0191d67df30e6c` |
| **Repositorio personal de documentación** | [`docs-r3-iam`, rama `docs/doc-import-workspace`](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam/tree/docs/doc-import-workspace) |
| **Repositorio documental del macroproyecto** | [`design-software-docs`, plan de mejoramiento](https://github.com/code-sena/design-software-docs/tree/main/11-quality/plan-mejoramiento) |
| **Documento de análisis relacionado** | [`validacion-alineacion-iam-develop-2026-10-23.md`](validacion-alineacion-iam-develop-2026-10-23.md) |

## 2. Revisión de la entrega de desarrollo

| Elemento revisado | Resultado | Observación basada en evidencia |
|---|---|---|
| Objeto de revisión identificado | ✅ | Es una alineación de ramas; no se documenta ni se presenta como HU. |
| Ramas y SHA identificados | ✅ | GitHub compara la rama de trabajo con `develop@c7735cf`. |
| Alineación con la referencia | ✅ | La rama de trabajo está 0 commits detrás de `develop`; el SHA de `develop` es ancestro del commit revisado. |
| Cambios R3 de observabilidad preservados | ✅ | Correlación, métricas y pruebas R3 permanecen en la rama de entrega. |
| CI del SHA revisado | ✅ | Los dos workflows consultados terminaron en `success` para `5350e14`. |
| Pruebas locales documentadas | ✅ con límite | `verify`: 201 aprobadas, 0 fallos/errores y 56 omitidas por falta de DB/broker. |
| Identidad de los commits de alineación y documentación | ✅ | GitHub asocia autor y committer de `33c2303` y `5350e14` al usuario Kevin indicado arriba. |
| Cambios publicados en un ambiente QA | ⏳ No verificado | No se encontró evidencia de despliegue o ejecución en un ambiente QA. |
| PR lista para fusión | ⏳ Pendiente | PR #43 aparece integrable, pero `mergeable_state=blocked` y no hay revisiones registradas. |
| Incidencias reproducidas en QA | No evaluable | No se ejecutaron pruebas funcionales en un despliegue QA; no se inventan bugs ni resultados de interfaz. |

La comparación registra **3 commits por delante y 0 por detrás**: el commit original de observabilidad R3, el merge de alineación y la actualización documental. La rama difiere de `develop` porque conserva el alcance R3 y sus documentos; eso no significa que le falten commits de la referencia.

## 3. Verificación de los cambios

### Cambio 1 — Incorporación de `develop`

**Documentado por Desarrollo:** incorporar a la rama R3 los cambios de referencia de endurecimiento de sesión y seguridad.

**Verificación:** el commit de merge [`33c2303`](https://github.com/code-sena/design-software-iam-service/commit/33c2303d9c004971e17c4a03eb63bb442f9f3188) tiene como padres el commit R3 `7c837e0` y `develop@c7735cf`. Su árbol es `6118d2264574f353af3f95caf394b3302d9723ff`, el mismo árbol del candidato probado localmente.

**Resultado:** ✅ Alineación de historial confirmada; la rama no está detrás de `develop`.

### Cambio 2 — Conservación de observabilidad R3

**Documentado por Desarrollo:** conservar correlación de solicitudes, métricas Prometheus y pruebas relacionadas al integrar la referencia.

**Verificación:** GitHub muestra en la comparación con `develop` los archivos R3 de filtro de correlación, `ApiErrors`, métricas, configuración y pruebas. La evidencia local registra aprobación de las pruebas de adaptadores y API relacionadas. No hay evidencia de scrape Prometheus o exportación OTLP en una infraestructura externa para este corte.

**Resultado:** ✅ Código y pruebas permanecen; ⏳ operación de observabilidad externa no verificada.

### Cambio 3 — Documentación de la integración

**Documentado por Desarrollo:** agregar el registro de validación, diferencias revisadas y evidencia local.

**Verificación:** el commit [`5350e14`](https://github.com/code-sena/design-software-iam-service/commit/5350e1495c7764a62cd47bc9e83063a7eeb68053) agrega/actualiza documentación y tiene como árbol `2cff118f33c7de1088f70c81e230ea3855d01efd`. El cambio es documental respecto del merge: no altera el árbol de código probado.

**Resultado:** ✅ Commit y árbol documental comprobados.

## 4. Verificación de commits

| Commit | Descripción | Autor/committer GitHub | Resultado QA |
|---|---|---|---|
| [`7c837e0`](https://github.com/code-sena/design-software-iam-service/commit/7c837e0824263cef0aef89f807731adca839f25e) | Implementación original de correlación y métricas R3 | `Kevin-Stiven-Lopez-Amaya404` | ✅ Se conserva como antecedente de la rama |
| [`33c2303`](https://github.com/code-sena/design-software-iam-service/commit/33c2303d9c004971e17c4a03eb63bb442f9f3188) | Merge de `develop` en la entrega R3 | `Kevin-Stiven-Lopez-Amaya404` | ✅ Identidad y dos padres verificados |
| [`5350e14`](https://github.com/code-sena/design-software-iam-service/commit/5350e1495c7764a62cd47bc9e83063a7eeb68053) | Documentación de alineación y evidencias | `Kevin-Stiven-Lopez-Amaya404` | ✅ Identidad y árbol verificados; es el HEAD actual consultado |

Los dos últimos commits fueron reconstruidos para corregir la atribución anterior. Los metadatos actuales de GitHub muestran tanto `author.login` como `committer.login` iguales a `Kevin-Stiven-Lopez-Amaya404`.

## 5. Verificación de correcciones y pruebas

No se documentó un bug independiente ni un fix aislado asociado a esta revisión. El cambio de código corresponde a la integración de la referencia `develop` y a la conservación de R3.

### Evidencia automatizada en la punta histórica del 23/10/2026

| Comprobación | Resultado | Evidencia |
|---|---|---|
| `IAM build and tests` | ✅ `success` | [GitHub Actions run 40053785643](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785643), SHA `5350e14` |
| `IAM R4 - Negative Security Tests` | ✅ `success` | [GitHub Actions run 40053785660](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785660), SHA `5350e14` |
| Maven local `verify` en el árbol integrado | ✅ 201 aprobadas; 0 fallos/errores | [`validacion-local.json`](evidencias-alineacion-iam-2026-10-23/validacion-local.json) y [`maven-verify-resumen.txt`](evidencias-alineacion-iam-2026-10-23/maven-verify-resumen.txt) |
| Casos negativos R4 | ✅ 20/20 presentes y aprobados localmente | Lista de resultados en [`validacion-local.json`](evidencias-alineacion-iam-2026-10-23/validacion-local.json) |
| Integración con PostgreSQL/broker | ⏳ No comprobada en esta ejecución | 56 pruebas de integración quedaron omitidas sin esas dependencias |

La CI verde se verificó para el SHA actual después de corregir los commits. Los resultados locales documentados corresponden al mismo árbol de código integrado, aunque se generaron antes de la reconstrucción de los metadatos de commit; la reconstrucción conservó el árbol (`6118d226`).

## 6. Verificación de dependencias

| Dependencia/elemento | Estado verificado |
|---|---|
| `develop` de IAM Service | ✅ Integrado como padre del merge `33c2303`; no queda atraso respecto de la referencia consultada. |
| Java/Maven local | ✅ Temurin 21.0.12.1 y Maven 3.9.10 documentados en la ejecución local. |
| PostgreSQL y esquema de prueba | ⏳ No disponibles/configurados en el `verify` informado; por ello se omitieron las IT. |
| RabbitMQ/broker | ⏳ No configurado en el `verify` informado; no se acredita el flujo del relay/outbox con broker real. |
| Colector y destinos de observabilidad | ⏳ Sin evidencia de conexión a Prometheus, Tempo u otro destino externo en esta revisión. |
| Dependencia de HU | No aplica; esta entrega no está asociada a una HU en este documento. |

## 7. Criterios de validación técnica

Estos criterios reemplazan los criterios de aceptación de HU de la plantilla; no son criterios de una historia de usuario.

| Criterio técnico | Evidencia disponible | Resultado |
|---|---|---|
| La rama contiene todo `develop` al corte revisado | Comparación GitHub: `behind_by=0`; base de comparación `c7735cf` | ✅ Cumple |
| La integración preserva el trabajo R3 | Árbol del merge y comparación de archivos; el candidato reporta las rutas R3 conservadas | ✅ Cumple |
| CI termina correctamente en el SHA actual | Dos workflows terminados en `success` sobre `5350e14` | ✅ Cumple |
| Pruebas locales cubren unitarias y los 20 casos negativos | 201 aprobadas; 20/20 negativos aprobados | ✅ Cumple con alcance local |
| Pruebas de integración con servicios reales pasan | 56 omitidas sin PostgreSQL/broker | ⏳ No comprobado |
| Contrato OpenAPI corresponde a la autorización implementada | El contrato declara `IDENTITY_USER_VIEW` en revocación; la implementación de `develop` usa `IDENTITY_USER_MANAGE` para sesiones ajenas | ⚠️ Diferencia pendiente de alineación |
| El comportamiento fue probado en ambiente QA desplegado | No se aportó URL, despliegue, evidencia de ejecución ni resultados de QA | ⏳ No comprobado |
| La PR cuenta con aprobación de revisión | PR abierta, `mergeable=true`, `mergeable_state=blocked`; reviews consultadas: ninguna | ⏳ Pendiente |

## 8. Diferencias encontradas

| Diferencia | Descripción | Estado QA |
|---|---|---|
| Contrato frente a autorización de revocación | `contracts/openapi/iam.yaml` declara `IDENTITY_USER_VIEW` para `DELETE` de sesión. El código de referencia requiere `IDENTITY_USER_MANAGE` para revocar sesiones de otra persona y permite autogestión. | ⚠️ Confirmada en el análisis; requiere acordar y sincronizar el contrato fuente y su copia institucional. No rebajar la autorización de código para ocultar el desfase. |
| Integración externa no ejecutada | Las pruebas locales omitieron 56 IT al no tener PostgreSQL/broker. | ⏳ Pendiente de evidencia con dependencias reales. |
| Validación operacional de observabilidad | No se verificaron scrape, exportación OTLP, ingesta de logs ni trazas en destinos externos. | ⏳ Pendiente si forma parte de la aceptación de entrega. |
| Estado de revisión PR | La PR es integrable, pero no cuenta con review registrada y GitHub la marca `blocked`. | ⏳ Requiere revisión/aprobación antes de declarar la PR lista para fusión. |

La existencia de estas diferencias no contradice que la rama ya esté al día con `develop`; describen pendientes de contrato, entorno o flujo de revisión.

## 9. Incidencias encontradas

No hay evidencia de un defecto reproducido en un ambiente QA, por lo que **no se registra un BUG ni se afirma un fallo funcional observado**. Se registran las observaciones verificables de la sección 8 como pendientes de revisión, no como bugs confirmados.

## 10. Revisión de la documentación de desarrollo

| Elemento | Estado | Observación |
|---|---|---|
| Alcance de la entrega | ✅ | Se identifica como alineación técnica de rama; no se atribuye a una HU. |
| Cambios realizados | ✅ | La documentación del servicio describe la integración R4 y la conservación R3. |
| Commits y autoría | ✅ | Los dos commits nuevos aparecen bajo la identidad Git del desarrollador indicado. |
| Evidencias de CI | ✅ | Enlaces a los dos workflows exitosos sobre el SHA `5350e14`. |
| Pruebas locales | ✅ con límite | Se declaran 201 aprobadas y 56 omitidas, sin presentarlas como E2E completo. |
| Dependencias externas | ✅ con límite | DB, broker y destinos de observabilidad figuran como no verificados. |
| Estado del contrato OpenAPI | ⚠️ | Se señala el desfase VIEW/MANAGE pendiente; no se presenta el contrato como alineado. |
| Estado de PR | ✅ | Se deja constancia de PR abierta, integrable, bloqueada y sin reviews. |

### Observaciones documentales

El informe previo [`validacion-alineacion-iam-develop-2026-10-23.md`](validacion-alineacion-iam-develop-2026-10-23.md) conserva un diagnóstico de corte anterior y contiene estados históricos de rama antes de la integración. Para el estado actual de commits y CI debe usarse este documento junto con la documentación publicada en la rama actual. No reutilizar del informe histórico afirmaciones de que la rama aún no fue publicada o que no hay CI sobre el candidato actual.

## 11. Resultado de la revisión (estado actualizado)

### Resultado

- ☑ **ALINEACIÓN DE HISTORIAL CON `develop` CONFIRMADA EN SERVICE, DB Y MFE**
- ☑ **CI SERVICE Y DB APROBADA EN SUS PUNTAS ACTUALES**
- ☐ **CI MFE / VALIDACIÓN QA INTEGRAL COMPLETADA**

### Motivo

La comparación remota confirma cero commits detrás de `develop` en las tres ramas de trabajo. Service y DB tienen CI exitosas en sus puntas actuales; MFE falla antes de instalar dependencias porque no está configurado `FRONTEND_DEPENDENCIES_READ_TOKEN`. La evidencia disponible tampoco contiene un smoke del conjunto de SHA actuales ni resuelve el desfase OpenAPI `VIEW`/`MANAGE`. Este resultado acredita alineación de historial, **no** el cierre de QA, ausencia de observaciones, aprobación de PR o certificación E2E.

## 12. Observaciones para Desarrollo

1. Solicitar a un administrador de MFE que configure `FRONTEND_DEPENDENCIES_READ_TOKEN` con acceso de lectura a los repositorios privados requeridos y vuelva a ejecutar CI.
2. Repetir el smoke MFE → Service → DB usando las puntas actuales enlazadas arriba; conservar comandos y resultados. Ejecutar PostgreSQL y RabbitMQ donde aplique.
3. Resolver con el dueño del contrato la diferencia de revocación en `design-software-shared-contracts`; luego actualizar el pin y regenerar los tipos afectados. No cambiar el permiso por inferencia.
4. Aportar evidencia del web-shell y, si forma parte de la aceptación, del scrape Prometheus y la exportación/ingesta de trazas y logs.
5. Obtener y registrar revisión técnica de las PR abiertas antes de fusionar. Los commits documentales actuales de los tres repositorios se verificaron con autor y committer Kevin.

## 13. Conclusión QA

> Al corte 24/10/2026, Service, DB y MFE están 0 commits detrás de sus respectivas ramas `develop`; PR #43, #79 y #10 siguen abiertas. Service y DB aprobaron sus workflows actuales. MFE no ejecutó instalación, pruebas ni build por falta de `FRONTEND_DEPENDENCIES_READ_TOKEN`. Sigue pendiente el smoke con los SHA actuales, la decisión del contrato `VIEW`/`MANAGE` y la evidencia de shell/observabilidad distribuida. La alineación de ramas está comprobada, pero **la entrega no está libre de observaciones y no se declara lista para cierre QA, fusión o producción**.

**Resultado final:**

> ☑ Alineación de historial con las ramas de referencia comprobada.

> ☐ Aprobación final de QA/producción — no emitida; hay bloqueos verificables pendientes.

### Evidencias y referencias

- [Repositorio de trabajo asignado: IAM Service, rama `r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev)
- [Repositorio personal de documentación: `docs-r3-iam`, rama `docs/doc-import-workspace`](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam/tree/docs/doc-import-workspace)
- [Repositorio documental del macroproyecto: `design-software-docs`](https://github.com/code-sena/design-software-docs/tree/main/11-quality/plan-mejoramiento)
- [Rama actual](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev)
- [Comparación con `develop`](https://github.com/code-sena/design-software-iam-service/compare/develop...r3-observability-delivery-dev)
- [PR #43](https://github.com/code-sena/design-software-iam-service/pull/43)
- [PR #79 — IAM DB](https://github.com/code-sena/design-software-iam-db/pull/79)
- [PR #10 — IAM MFE](https://github.com/code-sena/design-software-iam-mfe/pull/10)
- [Commit merge `33c2303`](https://github.com/code-sena/design-software-iam-service/commit/33c2303d9c004971e17c4a03eb63bb442f9f3188)
- [Commit documental `5350e14`](https://github.com/code-sena/design-software-iam-service/commit/5350e1495c7764a62cd47bc9e83063a7eeb68053)
- [CI: IAM build and tests](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785643)
- [CI: IAM R4 negative security tests](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785660)
- [CI Service en la punta actual](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215283)
- [CI R4 Service en la punta actual](https://github.com/code-sena/design-software-iam-service/actions/runs/40092215448)
- [CI DB en la punta actual](https://github.com/code-sena/design-software-iam-db/actions/runs/40092216422)
- [CI MFE detenida por secreto ausente](https://github.com/code-sena/design-software-iam-mfe/actions/runs/40092215804)
- [Evidencia local del plan de mejoramiento](evidencias-alineacion-iam-2026-10-23/README.md)
