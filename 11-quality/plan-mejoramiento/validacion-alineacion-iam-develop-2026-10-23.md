# Validación de alineación IAM: rama R3 frente a develop

> Seguimiento de publicación: la integración y este informe se publicaron en `r3-observability-delivery-dev`, commit [`9c57c7d`](https://github.com/code-sena/design-software-iam-service/commit/9c57c7d2ced84fd7e307e84e95d2eaea569c86d8), mediante el merge `03f01f4`. La comparación remota posterior confirmó cero commits pendientes de `develop@c7735cf`. El texto siguiente conserva el diagnóstico previo; el [registro publicado de entrega](https://github.com/code-sena/design-software-iam-service/blob/r3-observability-delivery-dev/docs/observability/validation.md) describe la aplicación. DB/broker y E2E siguen pendientes.

> Estado: 🟡 En revisión | Corte: 2026-10-23, America/Bogota
> Elaboración: revisión técnica asistida con Codex | Equipo: IAM
> Entrega prevista: 2026-10-24

## Contexto

Se compararon las ramas `develop` y `r3-observability-delivery-dev` de [code-sena/design-software-iam-service](https://github.com/code-sena/design-software-iam-service). Se inspeccionaron sus árboles y diferencias, implementación, observabilidad, contratos y pruebas. Además, se ensayó la integración en una copia temporal y se ejecutó Maven sobre el resultado.

**Dictamen:** la rama de trabajo todavía no incorpora el endurecimiento R4 de `develop`. Está **8 commits detrás y 1 delante**, con base común `6987e0f`. La corrección recomendada es integrar `develop` completo en la rama R3, conservar las capacidades R3 existentes y revisar la documentación de resultados. El ensayo de esa integración no produjo conflictos y aprobó **201 pruebas y los 20 controles negativos de seguridad**.

**No se certifica E2E completo ni entrega lista para desplegar:** las 56 pruebas de integración se omitieron por falta de DB/broker en esta ejecución. La rama original no se modificó ni se publicó; la integración validada existe únicamente en la copia temporal. Este informe deja los cambios listos para ejecutar y revisar, no los presenta como aplicados al remoto.

## Contenido

### 1. Referencias exactas y método de comparación

| Elemento | Valor comprobado |
|---|---|
| Referencia | `develop` → [`c7735cfc3135aad83091e2cfac5abe5809853c85`](https://github.com/code-sena/design-software-iam-service/tree/c7735cfc3135aad83091e2cfac5abe5809853c85). |
| Trabajo | `r3-observability-delivery-dev` → [`7c837e0824263cef0aef89f807731adca839f25e`](https://github.com/code-sena/design-software-iam-service/tree/7c837e0824263cef0aef89f807731adca839f25e). |
| Base común | `6987e0fa64628b5602969572d34a900f022ae891`. |
| Entrega R3 | [PR43](https://github.com/code-sena/design-software-iam-service/pull/43), abierta y sin fusionar al consultar. |
| Cambio ya incorporado a referencia | [PR44](https://github.com/code-sena/design-software-iam-service/pull/44), cerrada y fusionada; merge commit `c7735cf`. |
| Clon original inspeccionado | `C:/Users/PC/Desktop/design-software-iam-service/iam-integration`. Rama local R3, árbol limpio antes y después de la revisión. |
| Referencia local actualizada | `origin/develop` apunta a `c7735cf`; la rama local `develop` de ese clon aún apunta a `6987e0f`. **No utilizar el `develop` local antiguo como referencia.** |
| Copia temporal del ensayo | `C:/Users/PC/AppData/Local/Temp/iam-alignment-20261023-aa00b0f1`, rama `validation/integrate-develop`. |
| Árbol del candidato integrado | `6118d2264574f353af3f95caf394b3302d9723ff`, con merge preparado y sin commit. |

GitHub se consultó en ambos sentidos. Después se contrastaron los objetos Git locales con esos SHA, usando diferencias de árboles y la base común. Esto evita confundir el diff de una PR con todos los cambios actuales de `develop`.

La respuesta consultada de PR43 todavía indicaba `base_sha=6987e0f`, mientras que la comparación directa de ramas y `origin/develop` identificaban `c7735cf`. Para esta validación se utiliza **el SHA de la comparación directa**, no aquella base de la PR.

**Inventario:** hay **25 archivos distintos entre las puntas**: 10 pertenecen al avance de referencia y 15 a la ampliación R3. Al comparar cada rama con la base común, ambos conjuntos de archivos son disjuntos. No hay renombrados de módulos ni cambios en la estructura hexagonal: se conservan `iam-core`, `iam-adapters`, `iam-api` y `contracts`.

Los 8 commits exclusivos de referencia son `ae2be35`, `b5dd5af`, `7cfcfb1`, `b1905d5`, `4696fbe`, `e5c7666`, `91dae32` y `c7735cf`. Incluyen implementación, pruebas, CI, un documento QA agregado y retirado y el merge de PR44. **No son ocho cambios funcionales independientes.** El único commit exclusivo R3 es `7c837e0`.

### 2. Correcciones obligatorias para incorporar la referencia

Las rutas de esta sección son relativas a IAM Service. Los diez archivos deben quedar con el contenido de `develop@c7735cf`: el ensayo confirmó que pueden incorporarse juntos sin editar manualmente otros módulos. P0 indica una diferencia funcional de seguridad que bloquea declarar alineación; P1 indica pruebas o automatización necesarias para conservarla.

| ID / prioridad | Archivo afectado | Desajuste actual de R3 | Corrección exacta y verificación |
|---|---|---|---|
| A01 · P0 | `iam-core/src/main/java/co/sena/iam/application/usecase/GetCurrentUser.java` | R3 verifica JWT y usuario, pero no consulta el estado persistido de la sesión indicada por `sid`. No incorpora el rechazo de sesión inexistente, revocada, vencida o de otro usuario. | Incorporar `RefreshTokenRepository` y `Clock` y la comprobación de sesión de la referencia, antes de cargar la identidad. Reutilizar los puertos existentes. Verificar los cuatro casos negativos añadidos en `GetCurrentUserTest`. El efecto está limitado a los consumidores de este caso de uso; no acredita revocación inmediata en otros microservicios. |
| A02 · P0 | `iam-api/src/main/java/co/sena/iam/api/config/UseCaseConfiguration.java` | El bean sigue construyendo `GetCurrentUser` con tres dependencias. | Incorporar el cableado de cinco dependencias de referencia: sumar repositorio de refresh y reloj. Debe aplicarse junto con A01 y A06; copiar solo el caso de uso dejaría el constructor desalineado. Verificar arranque del contexto Spring. |
| A03 · P0 | `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/SessionController.java` | El listado y la revocación llaman directamente los casos de uso, sin el control de identidad y permiso agregado en `develop`. | Incorporar el control completo de referencia: identidad mediante Bearer/`GetCurrentUser`, autogestión del dueño y, para sesiones ajenas, `SYSTEM_ADMIN` global más autorización efectiva. Listado: `IDENTITY_USER_VIEW`; revocación: `IDENTITY_USER_MANAGE`, respetando scopes/overrides. Mantener respuestas y mapeo de sesiones. |
| A04 · P0 | `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/AuthExceptionHandler.java` | Falta el handler de `ForbiddenException` para el flujo de sesiones. | Incorporar el handler de referencia con HTTP 403 y código `FORBIDDEN`. Mantener el uso de `ApiErrors.traceId(request)`, compatible con la prioridad MDC que ya tiene R3. Verificar rechazos del controlador sin invocar acciones de sesión no autorizadas. |
| A05 · P0 | `iam-adapters/src/main/java/co/sena/iam/adapter/out/security/JwtAccessTokenVerifier.java` | R3 convierte `sub` y `sid` directamente a UUID; no contiene la comprobación explícita de ausencia/blancos de la referencia. | Incorporar el guard que rechaza claims requeridos ausentes/blancos mediante `UnauthenticatedException`; conservar verificación criptográfica, emisor y manejo previo de JWT/UUID inválidos. No sustituir la biblioteca ni crear un segundo verificador. |
| A06 · P1 | `iam-core/src/test/java/co/sena/iam/application/usecase/GetCurrentUserTest.java` | Falta el fixture de sesión/reloj y las regresiones nuevas de estado de sesión. | Incorporar la prueba y stub de referencia: sesión inexistente, revocada, vencida y de otro dueño; adaptar constructores de casos previos. Son cambios de la referencia, no una abstracción nueva propuesta por este informe. |
| A07 · P1 | `iam-adapters/src/test/java/co/sena/iam/adapter/in/rest/SessionControllerTest.java` | Solo conserva los cuatro casos previos de listado/revocación; no cubre la política R4 ni su nuevo cableado. | Incorporar los mocks/identidades y los 13 casos totales de referencia: sin token, acceso ajeno, admin sin grant, diferencia VIEW/MANAGE, DENY y dueño sin privilegios administrativos. Preservar los casos satisfactorios y 404 previos. |
| A08 · P1 | `iam-adapters/src/test/java/co/sena/iam/adapter/out/security/JwtAccessTokenTest.java` | Faltan los dos casos explícitos agregados para JWT sin firma `alg:none` y payload manipulado. | Incorporar ambos tests de referencia y conservar firma ajena, vencimiento, ausencia de token y demás regresiones existentes. No interpretar la ausencia del test como prueba automática de que el verificador anterior aceptaba esos tokens. |
| A09 · P1 | `iam-api/src/test/java/co/sena/iam/api/SessionPostgresIT.java` | Falta `logoutImmediatelyRejectsAccessToken`. La suite tiene seis casos frente a siete en referencia. | Incorporar el escenario login → `/auth/me`200 → logout204 → mismo access token rechazado401. Requiere PostgreSQL real para ejecutarse; estuvo omitido en el ensayo actual. |
| A10 · P1 | `.github/workflows/iam-r4-security.yml` | R3 no contiene el workflow específico de seguridad de referencia. | Incorporar el YAML de referencia: Java 21, `mvn -B -ntp -pl iam-adapters -am test`, comprobación de 20 casos negativos y publicación Surefire. Mantener el workflow R3 existente; cubren alcances distintos. |

Fuentes de código fijadas: [GetCurrentUser](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/iam-core/src/main/java/co/sena/iam/application/usecase/GetCurrentUser.java), [SessionController](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/SessionController.java), [JWT verifier](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/iam-adapters/src/main/java/co/sena/iam/adapter/out/security/JwtAccessTokenVerifier.java), [workflow R4](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/.github/workflows/iam-r4-security.yml).

### 3. Diferencias R3 frente a referencia: qué conservar y por qué

Estos quince archivos explican el commit propio de la rama R3. `develop` **no los eliminó ni reemplazó mediante R4**: no incorporaba esos cambios desde la base común. Por ello, su ausencia o contenido anterior en la referencia no obliga a borrar observabilidad para integrar los controles nuevos. La integración ensayada los conserva íntegros.

La alineación recomendada significa incorporar todo el avance de referencia manteniendo la funcionalidad de la rama de entrega. No significa identidad binaria entre ambos árboles. Si la aceptación exige expresamente árboles idénticos, esa condición debe confirmarse: eliminar los cambios R3 retiraría métricas/correlación y alteraría el comportamiento existente, contrario al criterio de preservación de esta solicitud.

| ID | Archivo R3 diferente o adicional | Acción y fundamento |
|---|---|---|
| O01 | `.dockerignore` | **Conservar.** Excluye Git, `target`, entornos y logs del contexto Docker. Referencia no incorpora un reemplazo. La construcción de imagen sigue pendiente de validar. |
| O02 | `.github/workflows/ci.yml` | **Conservar.** Ejecuta reactor `verify` y publica XML; el R4 verifica además presencia/resultado de negativos y no incluye `iam-api`. Deben ejecutarse los dos checks sobre el commit final. |
| O03 | `.mvn/wrapper/maven-wrapper.properties` | **Conservar.** Fija Maven 3.9.10 y hace utilizable el wrapper. No requiere cambiar el POM raíz. El workflow R4 usa el `mvn` del entorno y no entra en conflicto con ese archivo. |
| O04 | `README.md` | **Conservar descripción R3; actualizar el estado después de integrar.** Documentar R4 incorporado, ambos workflows y enlace a nueva validación. No presentar cifras históricas como ejecución del commit final. |
| O05 | `docs/observability/ADR-009-r3-observability.md` | **Conservar como ADR local.** Explica correlación/métricas. Referencia no contiene decisión sustitutiva. No renumerar ni sobrescribir ADR centrales durante esta alineación. |
| O06 | `docs/observability/integration.md` | **Conservar y reconciliar la versión compatible.** Mantener configuración de agente, DB y métricas; añadir validación posterior sin cambiar instrucciones no afectadas. |
| O07 | `docs/observability/validation.md` | **Conservar como histórico del 20 de octubre; añadir corte nuevo o enlace.** Los 186 casos y 54 IT aprobadas históricos pertenecen a R3 previo. El ensayo actual tiene 201 aprobadas y 56 IT omitidas. Nunca sustituir una ejecución por otra sin fecha/alcance. |
| O08 | `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/ApiErrors.java` | **Conservar la prioridad `trace_id` de MDC**, seguida de `X-Request-Id` y UUID. R4 no modifica este archivo; su nuevo handler lo reutiliza. Volver al contenido anterior retiraría una capacidad R3 sin necesidad. |
| O09 | `iam-adapters/src/main/java/co/sena/iam/adapter/in/rest/CorrelationIdFilter.java` | **Conservar.** Valida/genera/devuelve `X-Correlation-Id` y restaura MDC. No participa en la política de autorización de sesiones. |
| O10 | `iam-adapters/src/test/java/co/sena/iam/adapter/in/rest/ApiErrorsTest.java` | **Conservar sus tres regresiones.** Cubren la prioridad de traza propia de R3. |
| O11 | `iam-adapters/src/test/java/co/sena/iam/adapter/in/rest/CorrelationIdFilterTest.java` | **Conservar sus seis regresiones.** Cubren encabezados y limpieza/restauración MDC, incluso ante excepción. |
| O12 | `iam-adapters/src/test/java/co/sena/iam/adapter/in/rest/HealthControllerTest.java` | **Conservar pruebas R3 de respuesta/correlación.** La referencia no cambia el controlador de salud ni reemplaza esas comprobaciones. |
| O13 | `iam-api/pom.xml` | **Conservar `micrometer-registry-prometheus`.** Retirarlo quitaría el registro agregado por R3. El cableado R4 cambia otra clase, no este POM. |
| O14 | `iam-api/src/main/resources/application.yml` | **Conservar exposición Prometheus, etiqueta `service` e histograma HTTP.** Las propiedades DB, JWT, broker y puerto heredadas no deben reescribirse durante la alineación. |
| O15 | `iam-api/src/test/java/co/sena/iam/api/IamApiApplicationContextTest.java` | **Conservar verificación de contexto/métricas R3.** La ejecución combinada aprobó los cuatro casos API; no fue necesario reestructurar la configuración. |

### 4. Implementación, logging, métricas y trazas que deben quedar intactos

| Área | Resultado de comparación | Decisión de alineación y límite |
|---|---|---|
| Módulos y dependencias base | Mismos módulos y POM raíz; Java 21/Spring Boot 3.3.4. No hay reordenamiento arquitectónico requerido. | Mantener estructura y versiones. Incorporar únicamente el cableado de referencia y la dependencia Prometheus ya existente en R3. |
| Login, refresh, recuperación, usuarios y RBAC base | No aparecen diferencias nuevas de sus implementaciones entre las puntas fuera de los cinco archivos productivos A01–A05 y `ApiErrors` R3. | No reescribir casos de uso, repositorios o política central. Los endpoints que usan `GetCurrentUser` sí reciben el cambio deliberado de validación de sesión de referencia. |
| Logs | `iam-api/src/main/resources/logback-spring.xml` es idéntico entre ramas: `LogstashEncoder`, servicio y MDC. | Mantener configuración. R3 añade `correlation_id` mediante filtro, sin reemplazar `trace_id/span_id`. Un log JSON de tests no demuestra ingesta en Loki. |
| Métricas | Las diferencias están en O13–O15: registro Prometheus, exposición e histograma HTTP. | Mantener; el contexto y la presencia de series se probaron en la suite. No se acreditó scrape, dashboard, umbral o mejora de latencia en esta validación. |
| Trazas | `deploy/Dockerfile.api` es idéntico: agente OTel 2.8.0 y `-javaagent`. `logback-spring.xml` también permanece igual. | No duplicar instrumentación ni introducir otro SDK/agente. Maven no arrancó la imagen con el agente: exportación OTLP, propagación HTTP/JDBC y Tempo quedan pendientes. |
| Health/readiness | Controlador productivo sin cambios entre ramas. | Conservar comportamiento y pruebas. La omisión de dependencias externas en tests no demuestra disponibilidad del broker. |
| Contratos generados y esquema | `contracts/` no cambió entre ambas puntas. La referencia no añade migraciones DB. | No editar DTO generados ni crear DDL para resolver este desfase de ramas. El desajuste documental OpenAPI específico se registra a continuación. |

### 5. Desajuste heredado de la referencia y verificaciones pendientes

**Contrato de revocación de sesiones:** en ambas puntas, [`contracts/openapi/iam.yaml:276`](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/contracts/openapi/iam.yaml#L276) declara `x-required-feature: IDENTITY_USER_VIEW` para `DELETE /users/{id}/sessions/{session_id}`. El código de `develop`, que debe incorporarse, exige `IDENTITY_USER_MANAGE` para revocar sesiones ajenas, además de administrador global y autorización efectiva. La autogestión del dueño está permitida por el código.

Este es un **desfase entre contrato y comportamiento de referencia**, no una razón para rebajar el control MANAGE en R3. La corrección documental debe acordarse e incorporarse de forma sincronizada en el contrato fuente del servicio y su copia institucional, explicando dueño frente a administrador, VIEW frente a MANAGE y DENY. No editar únicamente el código generado ni introducir una política alternativa en la rama de entrega. La propia descripción de PR44 registra esta actualización como pendiente.

| Pendiente | Evidencia disponible | Qué falta comprobar antes de declarar el resultado |
|---|---|---|
| Autorización administrativa con PostgreSQL/RBAC reales | `SessionControllerTest` valida la política mediante mocks; PR44 reconoce la regresión administrativa como pendiente. | Admin global con VIEW para listar; VIEW sin MANAGE no revoca; MANAGE permitido revoca; DENY bloquea; dueño puede autogestionar. No asignar este resultado a los mocks. |
| Logout y estado de sesión persistido | A09 existe en referencia y candidato; hubo 7 tests de `SessionPostgresIT` omitidos en la ejecución actual. | Ejecutar contra base migrada y conservar XML del caso `logoutImmediatelyRejectsAccessToken` aprobado, además del resto de IT. |
| Claims `sub/sid` ausentes o blancos | Guard explícito de A05 comprobado por inspección. Los dos tests JWT añadidos cubren firma/payload, no prueban por sí mismos cada rama de ese guard. | Evidencia específica de esos escenarios si la aceptación los exige; no se afirma su ejecución por aprobar el agregado de JWT. |
| Integración de dependencias | Las 56 IT se omitieron al no suministrar DB/broker. | Ejecutar con PostgreSQL compatible; si RabbitMQ no está disponible, separar omisión del resto de resultados y mantener el límite explícito. |
| Trazas/logs/métricas externas y shell | Código y pruebas locales de componentes disponibles. | Runtime con agente, colector, scrape y destinos; navegador/shell si forman parte de la aceptación. No inferirlos de `verify`. |
| CI final de la rama integrada | Hay CI R4 aprobada para un commit anterior de esa entrega; aún no existe commit remoto de este candidato. | Publicar el commit revisado y obtener ambos checks aprobados sobre su SHA exacto. Revisar protecciones de rama si se requiere certificar checks obligatorios; no se consultaron. |
| Identidad estricta entre árboles | La petición exige preservar comportamiento y alinear con referencia; R3 contiene ampliaciones de observabilidad propias. | Si se exige eliminar esas ampliaciones, confirmar ese criterio explícito. La propuesta actual conserva R3 e incorpora íntegramente R4, sin crear abstracciones nuevas. |

No se considera prueba independiente de ejecución la descripción de una PR. Sus resultados históricos y pendientes se distinguen del ensayo nuevo respaldado por XML y logs de esta revisión.

### 6. Integración ensayada y pruebas efectivamente ejecutadas

#### 6.1. Ensayo reproducible sin tocar la rama original

Se clonó el repositorio local en la carpeta temporal indicada, sin hardlinks. Desde `7c837e0`, se ejecutó `merge --no-commit --no-ff c7735cf` en una rama temporal. Resultado: **merge automático sin conflictos**, con **10 archivos preparados**, 665 inserciones y 22 eliminaciones frente a la rama R3 inicial.

Se comprobó que:

- Los diez archivos R4 del candidato coinciden íntegramente con la referencia, incluidos sus cinco archivos productivos.
- No hay entradas sin resolver ni modificaciones productivas posteriores al ensayo; las pruebas no requirieron cambiar POM, forks ni fuentes.
- `git diff --cached --check` pasó.
- Los quince archivos de ampliación R3 se conservan, pues referencia no modifica sus rutas desde la base común.
- La rama original sigue en `7c837e0` y con árbol limpio. No se ejecutaron commit, push ni merge sobre ella.

#### 6.2. Maven y controles de seguridad

Entorno: **Maven 3.9.10 y Temurin 21.0.12.1**. Se utilizó la caché Maven existente en modo offline. `IAM_DB_DSN` e `IAM_BROKER_HOST` se dejaron sin configurar en el proceso de validación para no conectar accidentalmente dependencias del equipo.

```powershell
mvn '-Dmaven.repo.local=F:/tools/maven-repository' -o --batch-mode --no-transfer-progress verify
```

La primera ejecución, dentro del aislamiento, falló durante `testCompile` al no resolver clases principales que sí estaban generadas. La misma fuente pasó al repetir fuera del aislamiento, con autorización de ejecución; no se modificó código para resolverlo. No se atribuye ese primer resultado a un defecto funcional confirmado de la rama.

| Grupo comprobado en XML nuevos | Archivos XML | Registrados | Aprobados | Fallos / errores | Omitidos |
|---|---:|---:|---:|---:|---:|
| `iam-core` · Surefire | 21 | 96 | 96 | 0 / 0 | 0 |
| `iam-adapters` · Surefire | 15 | 101 | 101 | 0 / 0 | 0 |
| `iam-api` · Surefire | 2 | 4 | 4 | 0 / 0 | 0 |
| `iam-api` · Failsafe | 9 | 56 | 0 | 0 / 0 | 56 |
| **Total** | **47** | **257** | **201** | **0 / 0** | **56** |

`verify` terminó **BUILD SUCCESS**, código 0, el **2026-10-23 a las 20:51:14 -05:00**, duración 1:44. También empaquetó el JAR. Los 56 omitidos no son pruebas aprobadas contra PostgreSQL. Frente al informe previo de R3, hay un caso IT más en referencia, por el nuevo test de logout; por eso la suite registra 56 y no 55.

Se extrajeron los **20 nombres exigidos por el propio workflow R4** y se contrastaron con los XML nuevos: **20/20 presentes y aprobados**, ninguno omitido. Cubren JWT rechazado, usuario/sesión inválidos, acceso ajeno, grants/overrides y límites de centro. Ese control se reprodujo localmente; no se ejecutó GitHub Actions sobre un candidato aún no publicado.

#### 6.3. CI remota consultada

Se verificó `success` en los runs R4 [40012866652](https://github.com/code-sena/design-software-iam-service/actions/runs/40012866652) y [39950339939](https://github.com/code-sena/design-software-iam-service/actions/runs/39950339939), asociados a `4696fbe59314b65a8ea90aabe3559d92525deaeb`. Son evidencia de ese commit, no del candidato combinado. El endpoint consultado de runs por commit filtra ejecuciones de PR: no devolvió resultados para el merge `c7735cf`; **esto no prueba ausencia de todo workflow de otro tipo**.

#### 6.4. Evidencias conservadas de esta validación

- [Índice de anexos](evidencias-alineacion-iam-2026-10-23/README.md).
- [Resultado local por suite y los 20 controles](evidencias-alineacion-iam-2026-10-23/validacion-local.json).
- [Extracto Maven de la repetición satisfactoria](evidencias-alineacion-iam-2026-10-23/maven-verify-resumen.txt).
- [Comparaciones remotas y metadatos seleccionados](evidencias-alineacion-iam-2026-10-23/comparacion-remota.json).

Los logs completos, XML y JAR permanecen en la copia temporal. Los anexos seleccionan resultados y metadatos, sin copiar propiedades de entorno ni cuerpos HTTP. Las rutas originales están en el índice de anexos; el extracto Maven registra el SHA-256 del log fuente. La carpeta temporal es auxiliar y puede desaparecer; para una entrega duradera conservar los anexos y publicar artefactos de CI del commit final.

### 7. Procedimiento concreto para ejecutar la alineación

Estas instrucciones son una propuesta revisable; **no se han aplicado al clon original**. Ejecutarlas en un clon del repositorio oficial, con árbol limpio. Si `develop` cambia respecto a `c7735cf`, volver a revisar el diff y las pruebas: el ensayo actual está fijado a los SHA de la sección 1.

```powershell
# Dentro del repositorio oficial IAM Service.
git status --short
# Continuar solo si no hay cambios locales que se puedan mezclar con la entrega.
git fetch origin
git switch r3-observability-delivery-dev
git rev-parse HEAD
git rev-parse origin/develop
# Corte validado: HEAD=7c837e0... y origin/develop=c7735cf...

git merge --no-ff --no-commit origin/develop
git diff --cached --stat
git diff --cached --check
git ls-files -u
# En el corte revisado: 10 archivos de R4, sin entradas en conflicto.

# Java 21 y dependencias disponibles; ejecutar reactor completo.
./mvnw.cmd --batch-mode --no-transfer-progress verify
```

Revisar los XML, no solo el exit code. Repetir el control de los veinte nombres del workflow R4. Para verificar DB, proporcionar las variables `IAM_DB_DSN`, `IAM_DB_USERNAME`, `IAM_DB_PASSWORD` de una base compatible de pruebas mediante el entorno del proceso, sin guardarlas en Git. Si corresponde validar outbox/relay, configurar además el broker de pruebas.

La comprobación de integración puede ejecutarse sin repetir las unitarias ya aprobadas:

```powershell
./mvnw.cmd --batch-mode --no-transfer-progress `
  '-Dtest=!**/*' '-Dsurefire.failIfNoSpecifiedTests=false' verify
```

Ese comando sigue dependiendo de la configuración DB/broker: no declarar IT aprobadas si el XML las muestra omitidas. Ejecutar también los escenarios administrativos reales pendientes de la sección 5; los tests existentes no sustituyen esa evidencia.

Después de revisar la integración y sus resultados, registrar el merge y actualizar `README.md`/documentación de validación mediante un commit adicional si corresponde. Publicar la rama y revisar PR43 para que apunte al `develop` actual y ejecute ambos workflows. No hacer push de `develop` ni forzar la rama de entrega como parte de este procedimiento.

Para comprobar que el commit final contiene la referencia y conserva únicamente las ampliaciones previstas:

```powershell
git merge-base --is-ancestor origin/develop HEAD
# Debe finalizar con código 0.
git diff --name-status origin/develop...HEAD
# Debe mostrar las ampliaciones R3 y cualquier ajuste documental explícito revisado.
```

La lista de veinte controles está en el workflow y en el anexo JSON. Los ajustes de contrato/documentación compartidos se deben revisar como tales; no incorporar refactorizaciones o cambios funcionales adicionales para hacer pasar esta alineación.

### 8. Criterios para la revisión previa a la entrega

| Control | Estado de este informe | Condición pendiente en la rama real |
|---|---|---|
| Avance completo de `develop` incluido | Ensayado sin conflictos. | Aplicar integración y comprobar ascendencia de referencia en el commit final. |
| Seguridad R4 conservada | Código inspeccionado y 20 negativos aprobados en candidato. | Ejecutar/registrar ambos checks de CI sobre SHA final publicado. |
| Correlación, métricas y logging R3 preservados | Conservados en candidato; sus pruebas y contexto pasaron. | Mantener archivos O01–O15 y registrar la nueva validación en documentación. |
| Pruebas de reactor | 201 aprobadas en candidato, 0 fallos/errores. | Conservar resultados del commit final; no trasladar sus cifras a otro SHA sin validar. |
| PostgreSQL, permisos reales y logout persistido | Pendientes en esta ejecución: 56 IT omitidas. | Aprobar IT con DB y escenarios administrativos reales; separar omisión de broker si persiste. |
| Contrato de revocación | Desfase confirmado en referencia. | Alinear fuente OpenAPI institucional/servicio con MANAGE y autogestión según política acordada. |
| Runtime, imagen y observabilidad distribuida | No ejecutados en esta validación. | Evidencia específica si son requisitos de entrega. |

**Resultado de entrega de esta revisión:** diferencias identificadas por archivo, integración mínima ensayada y comprobación local satisfactoria. **La rama remota sigue desalineada mientras no se aplique esa integración; la validación integral del entorno sigue pendiente de las evidencias indicadas.**

## Referencias

- [Rama de referencia](https://github.com/code-sena/design-software-iam-service/tree/develop) y [rama de trabajo](https://github.com/code-sena/design-software-iam-service/tree/r3-observability-delivery-dev).
- [Comparación de entregas respecto de develop](https://github.com/code-sena/design-software-iam-service/compare/develop...r3-observability-delivery-dev).
- [PR43, observabilidad](https://github.com/code-sena/design-software-iam-service/pull/43) y [PR44, endurecimiento R4](https://github.com/code-sena/design-software-iam-service/pull/44).
- [Workflow R4 fijado a referencia](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/.github/workflows/iam-r4-security.yml).
- [Informe histórico IAM del 21 de octubre](informe-verificado-plan-mejoramiento-iam-2026-10-21.md). Sus estados y resultados conservan el corte original.
- [Anexos de la validación actual](evidencias-alineacion-iam-2026-10-23/README.md).
