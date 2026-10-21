<!-- RESUMEN-EJECUTIVO
agente: Jesús Ariel González Bonilla (PM + Arquitecto)
capacidad: arquitectura de micro-frontends (capa "por debajo" del mockup)
fase: diseño (UX/UI + arquitectura frontend)
estado: en revisión
dependencias_entrada: 09-microservices/service-catalog.md; 07-api/contracts/openapi/*.yaml; mockup-ddd/flows/*.md; 12-ux-ui/navigation-map.md
consumidores_siguientes: construcción del mockup completo; luego el frontend real (micro-frontends)
tldr: La experiencia se organiza por rol/funcionalidad (flows), pero POR DEBAJO el frontend se construye como un micro-frontend por dominio (espejo de los 9 microservicios) más un app-shell host que compone. Este doc mapea dominio→micro-frontend→pantallas y cómo los flujos componen varios MFE.
decisiones_clave: 1 micro-frontend por dominio (9) + 1 app-shell host; los flujos por rol son composiciones; cada MFE consume SU openapi.yaml; composición cross-dominio vía shell/read-models (BFF)
halts_registrados: ninguno
-->

# Arquitectura de Micro-Frontends (la capa "por debajo" del mockup)

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto)

Mapa de arquitectura frontend que sostiene los flujos. La topología describe el diseño; la
sección de integración IAM distingue la implementación y las pruebas disponibles de los
recorridos del host que siguen pendientes de comprobar.

## Principio (dos ejes que coexisten)

| Eje | Se organiza por | Artefacto |
|---|---|---|
| **Experiencia** (lo que ve el usuario) | **rol / funcionalidad** | `flows/*.md` (Coordinador, Instructor, …) |
| **Construcción** (cómo se implementa) | **dominio** | **un micro-frontend por microservicio** + **app-shell host** |

Un **flujo por rol** NO es un módulo de código: es una **composición** de pantallas/widgets
provistos por **varios micro-frontends**. Espeja el backend: 9 microservicios (dominio) →
9 micro-frontends (dominio) → compuestos en experiencias por rol.

## Componentes de la arquitectura

- **`shell-host`** (contenedor / orquestador): app shell (barra superior, notificaciones, menú de
  usuario), **navegación por rol** (derivada de `GET /auth/me.modules` + RBAC), enrutamiento,
  sesión/JWT, tema/design-system, estados globales (offline). **No** contiene lógica de dominio.
- **Un micro-frontend (MFE) por dominio**, dueño de las pantallas/componentes de SU dominio y que
  consume **su** `openapi.yaml`. El diseño prevé su montaje dentro del shell según la ruta/rol;
  la verificación local IAM descrita abajo no demuestra ese montaje.

> **Topología de despliegue (ADR-005).** El patrón es **hosts + remotes vía Module Federation**:
> - **Remotes** = los 9 MFE de dominio (responsive, deploy independiente). Slug **canónico** = el del
>   servicio: `reference-data-mfe`, `academic-management-mfe`, `training-environment-mfe` (renombrados
>   desde los cortos previos), más `iam/scheduling/actors/document/monitoring/audit`-mfe.
> - **Hosts** (agregadores que embeben los remotes): `web-shell` (fase 1) y `mobile-shell` (fase 2).
>   `shell-host` de este documento se materializa como estos hosts por experiencia.
> - **BFF por experiencia**: `web-bff` + `mobile-bff` (no un BFF único). Camino:
>   `cliente → gateway → BFF → servicios`.
> - **Notificaciones** = chrome global del host (campana/panel/inbox), consumiendo `notification-service`;
>   **no** hay `notification-mfe`.
>
> Las carpetas del mockup (`academic/`, `environment/`, `reference/`, …) conservan su nombre corto por
> ser artefacto ya construido; el mapeo a los repos reales usa el slug canónico de arriba.

## Integración IAM observada al 2026-10-21

La [PR MFE #10](https://github.com/code-sena/design-software-iam-mfe/pull/10), hacia `develop`,
sigue abierta y sin fusionar. La implementación revisada en
[`e73e844`](https://github.com/code-sena/design-software-iam-mfe/tree/e73e844bebebb64b99bfce24a908926f0e4cf76b)
expone `iamMfe/App` y `iamMfe/authentication`. Este último ofrece
`createAuthApi({ baseUrl, fetchImpl })`: el host puede inyectar su transporte instrumentado.
`App` acepta `AuthenticationPort` y el callback opcional `onAuthenticated(session)`, cuya
resolución espera la pantalla de login. El MFE entrega el `TokenResponse` contractual, sin
persistir los tokens; el host debe establecer la sesión y resolver navegación, refresh y logout.
Administración de usuarios y RBAC conservan sus puertos del host, sin probar aquí su integración
completa. Los indicadores UI de contraseña temporal son una extensión opcional del puerto;
no son campos que emita actualmente el backend.

El adaptador usa `/api/v1` por defecto y envía `{ email, password }`; consume tokens `snake_case`
sin wrapper `data`. Añade `X-Correlation-Id`, sin fabricar `traceparent`: la propagación de
contexto activo depende del transporte del host. El transporte standalone usa `fetch`; la
inyección disponible no acredita por sí sola la integración de la fachada del host exigida por
ADR-009. En desarrollo Vite sirve por defecto `5101` y reenvía `/api/v1` a `8080`; se configura
con `IAM_API_PROXY_TARGET` y `VITE_IAM_API_URL`. El smoke usó puertos temporales `15101` y `18080`.
El proxy de desarrollo no forma parte del build productivo ni valida una ruta edge desplegada.

Se conservan 46 pruebas jsdom aprobadas en nueve archivos y build federado local con
`remoteEntry.js` y tipos. El smoke cargó el adaptador mediante Vite y ejerció API/PostgreSQL;
comprobó sesión válida, correlación e identidad, con `/auth/me` 200 medido. Su resumen imprime
`login:200` como literal, sin conservar el estado HTTP medido del login. No montó el web-shell
en navegador ni demostró exportación de spans, almacenamiento/cookies del host o consumidores.
La [CI MFE](https://github.com/code-sena/design-software-iam-mfe/actions/runs/39540502857)
falló antes de instalar/probar por falta de `FRONTEND_DEPENDENCIES_READ_TOKEN` para las dos
dependencias privadas fijadas; los resultados locales no constituyen una CI aprobada.

Fuentes y límites: [informe técnico](../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md),
[matriz de evidencias](../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md)
y [anexos](../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/evidencias/README.md).

## Mapa dominio → micro-frontend → pantallas (dueño) → microservicio

| Micro-frontend | Microservicio (backend) | Pantallas/componentes que posee | Flujos donde aparece |
|---|---|---|---|
| **shell-host** | — (usa iam para sesión) | App shell, nav por rol, notificaciones (contenedor), páginas de error/offline | todos |
| **iam-mfe** | iam-service | Login, Recuperar/Nueva contraseña, Administración de **Usuarios/roles** | 01-auth, 05-administrador |
| **scheduling-mfe** | scheduling-service | **Horarios (lista)**, **Crear/editar horario**, **Panel de conflictos**, vistas **Mi horario** (semana) | 02-coordinador, 03-instructor, 04-aprendiz |
| **academic-management-mfe** | academic-management-service | **Fichas** (lista/consulta), programas/competencias | 02-coordinador |
| **training-environment-mfe** | training-environment-service | **Disponibilidad de ambientes**, gestión de ambientes/reservas | 02-coordinador |
| **actors-mfe** | actors-service | Instructores/aprendices, **Mi disponibilidad** (instructor) | 02-coordinador, 03-instructor |
| **document-mfe** | document-service | **Documentos**, **Plantillas de documento** | 06-backoffice |
| **monitoring-mfe** | monitoring-service | **Panel de indicadores (KPIs)**, **Seguimiento de ficha** | 05-administrador, 03-instructor, 04-aprendiz |
| **audit-mfe** | audit-service | **Auditoría** (traza) | 06-backoffice |
| **reference-data-mfe** | reference-data-service | **Parametrización / catálogos**, **Datos de referencia** (regionales/centros/sedes) | 05-administrador, 06-backoffice |

## Parametrización (07 · director + soporte): pantalla nueva por MFE

Las pantallas de parametrización configuran los catálogos y datos maestros que los flujos operativos
exigen como prerequisito. Cada MFE afectado suma **una** pantalla de parametrización:

| Micro-frontend | Pantalla de parametrización que suma | Prerequisito que habilita |
|---|---|---|
| **reference-data-mfe** | **Hub de parametrización** (`/admin/parametrizacion`, montado por el shell) + **Geografía institucional** | punto de entrada; regionales/centros/sedes |
| **academic-management-mfe** | **Currículo académico** | currículo → fichas |
| **scheduling-mfe** | **Jornadas / franjas horarias** | franjas → horarios |
| **training-environment-mfe** | **Tipos de ambiente e inventario** | tipos de ambiente → ambientes |
| **monitoring-mfe** | **Catálogos de monitoreo (KPI/alertas)** | catálogos → KPIs |
| **actors-mfe** | **Estados de actores** | máquina de estados de actores |
| **iam-mfe** | **RBAC — roles y permisos** | roles/permisos RBAC |

## Realización en el mockup

El mockup adoptado (`mockups/app/`) representa esta arquitectura como SPA estática con hash-router;
su login es simulado y no consume backend. Sus pantallas y capturas son referencias de diseño,
no evidencia del montaje federado o de autenticación real:

- **`shell/`** = `shell-host`: router (`app.js`), `routes.js`, `screens.js`, `shell.js` — marco,
  navegación por rol, guards RBAC y estados globales. No contiene lógica de dominio.
- **Una carpeta por dominio**, cada una dueña de su `screens.js`:
  `iam/ scheduling/ academic/ environment/ actors/ document/ monitoring/ audit/ reference/`.
- **`assets/` compartido** (un solo set de tokens/componentes: `tokens.css`, `components.css`,
  `app.css`, `components.js`, `icons.js`) — ningún dominio redefine tokens.
- **`tools/validate-routes.js`** valida la cobertura ruta↔pantalla (**53/53**).

## Cómo un flujo compone varios micro-frontends

Ejemplo — **Dashboard del Coordinador** (una sola pantalla, varios MFE):
```
shell-host (marco + nav + saludo)
├─ scheduling-mfe   → widget "Conflictos pendientes" + widget "Horarios en borrador"
├─ academic-management-mfe     → KPI "Fichas activas"
└─ (navegación)     → "Crear horario" abre una pantalla de scheduling-mfe
```
La composición cross-dominio (juntar datos de varios MFE en una pantalla) se resuelve con:
- **Composición en el shell** (el host monta widgets de cada MFE), y/o
- una capa **BFF / read-model** que agrega los datos que una pantalla necesita de varios dominios
  (ver `discovery-findings.md` — nombres de instructor/ambiente/competencia viven en otros dominios;
  referencias por UUID sin FK física, patrón de microservicios).

## Reglas de la arquitectura

1. **Cada MFE consume SOLO su `openapi.yaml`.** No llama directo a la BD ni a otro dominio; los
   datos de otro dominio llegan vía su MFE (composición) o vía BFF/read-model.
2. **El shell-host no tiene lógica de dominio** (solo marco, sesión, nav, tema).
3. **Design-system compartido** (tokens/componentes base) entre todos los MFE para coherencia
   visual (ver `12-ux-ui/design-system.md`). Un MFE no redefine tokens.
4. **Contract-first:** el MFE se construye contra el contrato (no contra el mockup — el frontend
   real jamás conoce el mockup; ver `README.md`).
5. **Trazabilidad:** cada pantalla del flujo declara su **MFE dueño** además de `HU → endpoint →
   tabla` (se agrega el campo "MFE" al header de cada pantalla en la maduración del spec).

## Relación con el resto del spec

- Los **flujos** (`flows/*.md`) siguen siendo la unidad de **experiencia** (y de generación de
  mockup por Stitch/GPT).
- Este documento es la unidad de **arquitectura de construcción** (qué MFE posee qué pantalla).
- La **maduración del spec** conecta ambos: cada pantalla del flujo apunta a su MFE dueño, y el
  inventario de pantallas/componentes se agrupa por MFE para la fase de construcción del frontend.
