# Índice de diagramas

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
>
> Diagramas de secuencia de `iam-service` y `notification-service`, expresados
> como bloques `mermaid` en Markdown dentro de
> [`diagrams/source/`](diagrams/source/). La carpeta [`diagrams/exports/`](https://github.com/code-sena/design-software-docs/tree/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/08-uml/diagrams/exports/)
> (PNG/SVG exportados) sigue preparada; al exportar, registrar la ruta en la columna correspondiente.
> Al agregar un diagrama, registrarlo como una fila en la tabla de abajo.

| Diagrama | Tipo | Archivo fuente | Exportación | Estado |
|----------|------|---------------|-------------|--------|
| Autenticación IAM — adaptador/proxy/API/DB y contrato pendiente del host | Secuencia | [`diagrams/source/login-jwt.md`](diagrams/source/login-jwt.md) | _(pendiente)_ | 🟡 Revisión; prueba HTTP local, shell y trazas sin acreditar |
| Outbox → relay → `iam-events` (`iam-service`) | Secuencia | [`diagrams/source/outbox-relay-events.md`](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/08-uml/diagrams/source/outbox-relay-events.md) | _(pendiente)_ | 🟢 |
| Consumo de notificación — entrega / retry / DLQ (`notification-service`) | Secuencia | [`diagrams/source/notification-consume.md`](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/08-uml/diagrams/source/notification-consume.md) | _(pendiente)_ | 🟢 |

El diagrama IAM incorpora la entrega R3 en PR abiertas y distingue el recorrido
local comprobado del callback/inyección del host aún sin montaje en navegador.
Los resultados históricos y pendientes están en el
[expediente IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/README.md).
Los estados de los otros diagramas conservan su alcance previo: no se revalidaron
sus recorridos mediante esta actualización documental.
