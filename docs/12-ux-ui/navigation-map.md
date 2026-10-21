# Mapa de Navegación

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: UX-UI

Estructura de navegación **prevista** de la plataforma SENA — Gestión de Horarios, organizada por rol de negocio.

> **Estado observado de IAM al 2026-10-21:** el MFE implementa login por correo y entrega la sesión a `onAuthenticated(session)` del host. Tiene 46 pruebas jsdom y build aprobados localmente; la PR #10 está abierta y su CI está bloqueada por la credencial de dependencias privadas. La composición con el web-shell, el aterrizaje por rol y la navegación protegida no se demostraron en navegador. Este mapa conserva la jerarquía de navegación planificada; el [informe IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md) y la [matriz de evidencias](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) delimitan lo implementado y probado.

## Roles y punto de entrada

Roles descritos por este mapa: `COORDINATOR`, `INSTRUCTOR`, `LEARNER`, `ADMIN` ([discovery M1](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/discovery-brief.md)); contrastar el catálogo vigente con el contrato IAM. El diseño prevé que, tras autenticarse (`POST /api/v1/auth/login` → `TokenResponse`), el shell establezca la sesión y resuelva el espacio del rol. El MFE entrega la respuesta contractual al callback; esta entrega no demuestra la redirección ni el menú real del shell.

## Áreas de navegación por rol (previstas)

### Coordinador Académico (rol central del MVP)

Responsable de crear, validar y publicar horarios. Navegación prevista:

- **Inicio / resumen** — fichas activas del centro, horarios en borrador, conflictos pendientes.
- **Horarios**
  - Lista de horarios por ficha y estado (BORRADOR / EN_REVISIÓN / PUBLICADO).
  - Crear / editar horario → agregar sesiones (ficha + instructor + ambiente + franja + fecha).
  - Validar conflictos → panel de conflictos.
  - Publicar / versionar.
- **Disponibilidad** — consulta de ambientes e instructores disponibles por franja.
- **Fichas** — consulta de fichas del centro y su programa.

### Instructor

Consulta su carga y (en evolución) su disponibilidad.

- **Mi horario** — vista semanal de sus sesiones de clase.
- **Mi disponibilidad** — gestión de franjas disponibles (según alcance por fase).
- **Seguimiento** — registro de sesiones de seguimiento de ficha (asistencia, avance).

### Aprendiz

Consulta de su horario vigente.

- **Mi horario** — vista semanal de clases de su ficha.
- **Notificaciones** — avisos de cambios de horario.

### Administrador de Centro

Visibilidad y métricas.

- **Panel de indicadores** — utilización de ambientes, carga de instructores, KPIs de seguimiento ([discovery M9](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/discovery-brief.md)).
- **Administración** — gestión de datos de referencia y usuarios del centro (según permisos).

## Diagrama de navegación previsto (alto nivel)

```
                         [ Login / Autenticación ]
                                    │ JWT (rol)
        ┌───────────────┬───────────┴───────────┬────────────────┐
        ▼               ▼                       ▼                ▼
   COORDINADOR      INSTRUCTOR              APRENDIZ           ADMIN
   ───────────      ──────────              ────────          ─────
   Inicio           Mi horario              Mi horario        Indicadores
   Horarios ─┐      Mi disponibilidad       Notificaciones    Administración
   Disponib. │      Seguimiento
   Fichas    │
             └─▶ Crear → Sesiones → Validar (conflictos) → Publicar → Versionar
```

## Reglas de navegación

1. **Segmentación por rol:** el JWT determina las áreas visibles; no se muestran rutas fuera del rol.
2. **El estado guía el flujo:** desde un horario en BORRADOR se llega a Validar y Publicar; un PUBLICADO es de solo lectura y ofrece "crear nueva versión".
3. **Rutas críticas cortas:** el flujo Crear → Validar → Publicar es el camino principal del coordinador y debe minimizar saltos.
4. **Ámbito por centro:** el coordinador opera dentro de su centro de formación ([discovery M2](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/discovery-brief.md)).

## Pendiente de diseño e integración

- [ ] Validar el árbol de rutas/URLs del shell real frente al [sitemap](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/mockup-ddd/sitemap.md) y a las rutas ya implementadas del MFE.
- [ ] Comprobar en navegador la entrega de sesión al shell, el aterrizaje por rol y la navegación protegida.
- [ ] Especificar breadcrumbs y navegación secundaria por área.
- [ ] Validar el mapa con usuarios reales (coordinadores) antes de wireframear.

## Referencias

- [wireframes.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/wireframes.md) · [design-system.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/design-system.md)
- [discovery-brief.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/discovery-brief.md) · [overview.md (flujos)](../05-architecture/overview.md#flujos-principales)
