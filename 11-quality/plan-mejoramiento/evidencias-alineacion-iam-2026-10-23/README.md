# Evidencias de alineación de ramas IAM

> Estado: 🟡 En revisión | Fecha: 2026-10-23, America/Bogota
> Elaboración: validación técnica asistida con Codex | Equipo: IAM

## Contexto

Anexos del [informe de validación](../validacion-alineacion-iam-develop-2026-10-23.md). Se conservaron resultados de una ejecución nueva sobre la integración temporal de R3 `7c837e0` y referencia `c7735cf`, sin modificar ni publicar la rama original.

## Contenido

| Archivo | Procedencia y alcance |
|---|---|
| [validacion-local.json](validacion-local.json) | Recuento directo de 47 XML y resultado de los veinte casos exigidos por el workflow R4. Incluye versiones, SHA, árbol candidato y límites. No contiene properties ni salidas internas de las suites. |
| [maven-verify-resumen.txt](maven-verify-resumen.txt) | Selección de resúmenes Maven, con números de línea del log original de la repetición satisfactoria. Incluye hash del original. No copia el log completo de aplicación. |
| [comparacion-remota.json](comparacion-remota.json) | Respuestas de comparación en ambos sentidos y selección de metadatos PR43/PR44 y runs R4, consultados durante la revisión. Los estados remotos son un corte, no información en vivo. |

Los originales están bajo `C:/Users/PC/AppData/Local/Temp/iam-alignment-20261023-aa00b0f1/`:

- `alignment-maven-verify.log`: primer intento dentro del aislamiento, fallido en `testCompile`.
- `alignment-maven-verify-retry.log`: repetición fuera del aislamiento, `BUILD SUCCESS`, sin cambios de fuente/POM.
- `iam-core/target/surefire-reports/`, `iam-adapters/target/surefire-reports/`, `iam-api/target/surefire-reports/` y `iam-api/target/failsafe-reports/`: XML de la ejecución.
- `iam-api/target/iam-api-0.1.0.jar`: artefacto local empaquetado; no imagen Docker ni despliegue validado.

El candidato conserva un merge preparado sin commit, árbol Git `6118d2264574f353af3f95caf394b3302d9723ff`. El árbol original de la rama R3 siguió limpio. Una carpeta temporal no es almacenamiento permanente: conservar estos anexos y los artefactos CI del commit final para entrega.

Resultados: **201 aprobadas, 56 omitidas, 0 fallos/errores; 20/20 negativos aprobados**. Las 56 IT no conectaron PostgreSQL/broker, por lo que este resultado no acredita E2E ni runtime de observabilidad.

## Referencias

- [Informe de validación](../validacion-alineacion-iam-develop-2026-10-23.md).
- [Workflow que define los veinte casos](https://github.com/code-sena/design-software-iam-service/blob/c7735cfc3135aad83091e2cfac5abe5809853c85/.github/workflows/iam-r4-security.yml).
