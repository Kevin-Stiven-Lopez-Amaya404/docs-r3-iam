# Revisión QA de entrega técnica — alineación IAM (sin HU)

> Este documento sigue como referencia la estructura de `Plantilla_qa.md`, pero el objeto revisado es una entrega técnica de alineación entre ramas. **No corresponde a una Historia de Usuario** y no se asignan criterios, bugs ni dependencias de HU.

## 1. Información general

| Campo | Información |
|---|---|
| **Objeto de revisión** | Alineación de `r3-observability-delivery-dev` con `develop` en IAM Service |
| **Tipo de entrega** | Integración de cambios de referencia, preservación de observabilidad R3 y documentación de validación |
| **Identidad Git de los commits de esta entrega** | `Kevin-Stiven-Lopez-Amaya404` (`mrroobot08@gmail.com`), confirmada por los metadatos de GitHub |
| **Responsable QA** | No indicado en la evidencia disponible |
| **Fecha de corte** | 23/10/2026, America/Bogota |
| **Ambiente revisado** | GitHub (rama, PR y GitHub Actions) y ejecución Maven local documentada |
| **Asignación individual** | Kevin Stiven López Amaya: R3 de observabilidad en IAM Service, según el Acta 02 y el Anexo de Aporte Individual revisados |
| **PR** | [#43 — feat(iam): integrate request correlation and Prometheus metrics](https://github.com/code-sena/design-software-iam-service/pull/43) |
| **Repositorio de trabajo asignado** | [`code-sena/design-software-iam-service`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev), rama `r3-observability-delivery-dev` |
| **Rama de referencia** | [`develop`](https://github.com/code-sena/design-software-iam-service/tree/develop), SHA `c7735cfc3135aad83091e2cfac5abe5809853c85` |
| **Rama revisada** | [`r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev), SHA `5350e1495c7764a62cd47bc9e83063a7eeb68053` |
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

### Evidencia automatizada en el SHA actual

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

## 11. Resultado de la revisión

### Resultado

- ☑ **APTO PARA CONTINUAR A VALIDACIÓN QA, CON OBSERVACIONES PENDIENTES**
- ☐ **DEVUELTO A DESARROLLO POR FALLA DE ALINEACIÓN CON `develop`**

### Motivo

La alineación con `develop` está comprobada, la identidad de los commits fue corregida y los dos workflows del SHA actual aprobaron. Se permite continuar con la validación QA. Esto **no equivale a aprobación de la PR, aprobación de producción ni certificación E2E**: la PR no tiene revisión aprobatoria, 56 pruebas de integración fueron omitidas y el desfase OpenAPI continúa pendiente.

## 12. Observaciones para Desarrollo

1. Solicitar y registrar la revisión técnica pendiente en PR #43 antes de fusionar.
2. Acordar y sincronizar el contrato OpenAPI de revocación de sesiones con la política implementada (`IDENTITY_USER_MANAGE` para sesiones ajenas y autogestión del dueño), incluyendo la copia institucional correspondiente.
3. Ejecutar las pruebas de integración con PostgreSQL y, para el relay, RabbitMQ; adjuntar resultados del SHA que se valide. Mantener explícitas las pruebas que continúen omitidas.
4. Si la aceptación exige observabilidad operacional, aportar evidencia del runtime con scrape Prometheus y exportación de trazas/logs a los destinos configurados.
5. Mantener el autor y committer de los siguientes cambios bajo la identidad Git esperada y revisar los metadatos de GitHub antes de entregar.

## 13. Conclusión QA

> La revisión documental y de GitHub confirma que `r3-observability-delivery-dev` contiene `develop` al SHA consultado, conserva la ampliación R3, tiene sus dos workflows actuales exitosos y muestra los commits de integración/documentación bajo la identidad Git de Kevin. La evidencia no demuestra despliegue en QA ni ejecución de las 56 pruebas de integración omitidas; además, queda pendiente alinear el contrato OpenAPI y obtener aprobación de la PR. La entrega puede continuar a validación QA con esos límites visibles, pero no se declara libre de observaciones ni lista para fusión o producción.

**Resultado final:**

> ☑ Alineación con la rama de referencia comprobada; continuar la validación QA con observaciones pendientes.

> ☐ Aprobación final de QA/producción — no emitida en este documento.

### Evidencias y referencias

- [Repositorio de trabajo asignado: IAM Service, rama `r3-observability-delivery-dev`](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev)
- [Repositorio personal de documentación: `docs-r3-iam`, rama `docs/doc-import-workspace`](https://github.com/Kevin-Stiven-Lopez-Amaya404/docs-r3-iam/tree/docs/doc-import-workspace)
- [Repositorio documental del macroproyecto: `design-software-docs`](https://github.com/code-sena/design-software-docs/tree/main/11-quality/plan-mejoramiento)
- [Rama actual](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev)
- [Comparación con `develop`](https://github.com/code-sena/design-software-iam-service/compare/develop...r3-observability-delivery-dev)
- [PR #43](https://github.com/code-sena/design-software-iam-service/pull/43)
- [Commit merge `33c2303`](https://github.com/code-sena/design-software-iam-service/commit/33c2303d9c004971e17c4a03eb63bb442f9f3188)
- [Commit documental `5350e14`](https://github.com/code-sena/design-software-iam-service/commit/5350e1495c7764a62cd47bc9e83063a7eeb68053)
- [CI: IAM build and tests](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785643)
- [CI: IAM R4 negative security tests](https://github.com/code-sena/design-software-iam-service/actions/runs/40053785660)
- [Evidencia local del plan de mejoramiento](evidencias-alineacion-iam-2026-10-23/README.md)
