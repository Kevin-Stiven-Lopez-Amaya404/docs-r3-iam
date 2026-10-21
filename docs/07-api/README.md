# API

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Autor: Kevin Andrey Culma (Arquitecto) | Equipo: Arquitectura.

## Contenido

Define lineamientos de API, autenticación y ubicación de contratos OpenAPI.

> **Estado observado de IAM al 2026-10-21:** existe API implementada y se revisó la entrega de correlación/métricas de [Service #43](https://github.com/code-sena/design-software-iam-service/pull/43), los controles de compatibilidad de [DB #79](https://github.com/code-sena/design-software-iam-db/pull/79) y la alineación del consumidor de [MFE #10](https://github.com/code-sena/design-software-iam-mfe/pull/10). Las tres PR siguen abiertas. Service y DB tienen CI aprobada; MFE tiene 46 pruebas y build locales aprobados, con CI bloqueada por acceso a dependencias privadas. El smoke adaptador→proxy→API→PostgreSQL no valida el shell ni el recorrido con consumidores. Consultar el [expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md) para fuentes y límites; el estado del resto de las APIs debe contrastarse por servicio.

## Archivos

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [guidelines.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/guidelines.md) | Convenciones de diseño de APIs | 🟡 |
| [authentication.md](./authentication.md) | Estrategia de autenticación y autorización | 🟡 |
| [api-surface-map.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/api-surface-map.md) | Inventario de endpoints detectados por servicio | 🟡 |
| [openapi-contract-review.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/openapi-contract-review.md) | Revisión de cobertura entre código y contratos OpenAPI | 🟡 |
| [Informe del plan IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md) | Implementación, pruebas y límites de integración contrastados | 🟡 |
| [contracts/openapi/](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/contracts/openapi/) | Contratos OpenAPI validados y publicables | 🔴 |

## Dónde va cada contrato

| Tipo de contrato | Dónde vive | Cuándo se usa |
|-----------------|------------|---------------|
| Contrato de implementación (borrador, en evolución) | `09-microservices/services/<svc>/api-contract.md` | Durante el desarrollo del servicio |
| Contrato OpenAPI publicable (estable, revisado) | `07-api/contracts/openapi/<svc>.yaml` | Cuando el contrato es estable y puede compartirse con consumidores externos |

**Regla:** el contrato en `07-api/contracts/openapi/` es la versión aprobada por arquitectura. Si hay diferencia entre ambos, el de `07-api/` tiene precedencia para consumidores externos.

## Plantillas

| Plantilla | Descripción |
|-----------|-------------|
| [_template-api-contract.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/07-api/_template-api-contract.md) | Contrato REST: autenticación, endpoints CRUD, errores, rate limiting y versionado |
