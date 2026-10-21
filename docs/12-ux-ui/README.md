# UX/UI

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: UX-UI

## Contenido

Documenta sistema de diseño, navegación y wireframes del producto.

> **Estado observado de IAM al 2026-10-21:** existe implementación del micro-frontend con login por correo, puerto de autenticación y entrega de sesión al host. La [PR MFE #10](https://github.com/code-sena/design-software-iam-mfe/pull/10) sigue abierta: 46 pruebas jsdom y build federado aprobados localmente; CI se detuvo por ausencia de `FRONTEND_DEPENDENCIES_READ_TOKEN`. El smoke HTTP ejerció adaptador, proxy, API y DB; no montó el web-shell en un navegador. El [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) conserva el alcance, los resultados y las limitaciones.

`design-system.md` sigue siendo la guía de referencia; `navigation-map.md` describe la navegación prevista por rol. Los [flujos y su arquitectura MFE](./mockup-ddd/micro-frontends.md) distinguen diseño, prototipo estático e implementación comprobada. Las imágenes del mockup no son evidencia de integración con el backend.

## Archivos

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [design-system.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/design-system.md) | Tokens, componentes, patrones visuales y reglas UX/UI | 🟡 |
| [navigation-map.md](./navigation-map.md) | Mapa de navegación y jerarquía de pantallas | 🟡 |
| [wireframes.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/wireframes.md) | Wireframes y referencias de interacción | 🟡 |
| [mockup-ddd/micro-frontends.md](./mockup-ddd/micro-frontends.md) | Composición por dominio y alcance de integración IAM comprobado | 🟡 |
| [Expediente del plan IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) | Informe, matriz de pruebas y anexos de la entrega en revisión | 🟡 |

## Plantillas

| Plantilla | Descripción |
|-----------|-------------|
| [_template-ux-flows.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/_template-ux-flows.md) | Flujos UX: actores, happy path, caminos de error y diagramas |
| [_template-ui-spec.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/_template-ui-spec.md) | Spec de UI: pantallas, componentes, estados y accesibilidad |
| [_template-design-system.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/12-ux-ui/_template-design-system.md) | Design system: paleta, tipografía, espaciado, componentes e iconografía |
