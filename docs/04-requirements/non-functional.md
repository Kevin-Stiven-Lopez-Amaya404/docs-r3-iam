# Requisitos No Funcionales (NFR)

> Estado: 🟡 En revisión | Última actualización: 2026-10-21
> Fuente: [problem-framing.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/03-product/problem-framing.md) · [overview.md](../05-architecture/overview.md)
> Cada NFR es verificable y tiene una métrica objetivo.

> **Corte de evidencia IAM: 2026-10-21.** Las marcas «✅ Evidencia dev» conservan antecedentes declarados por los servicios; no certifican que cada requisito haya sido repetido en este plan. La entrega IAM revisada tiene 186 casos aprobados en CI Service y 54 integraciones PostgreSQL locales aprobadas, más una omitida; MFE conserva 46 pruebas jsdom y build locales aprobados, con CI bloqueada por acceso a dependencias privadas. Las PR siguen abiertas: no se acredita promoción de esta entrega a `develop`, QA, staging o producción. El [informe IAM](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/informe-tecnico.md) y su [matriz](../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md) relacionan criterios con pruebas y límites. El resto de los requisitos conserva su métrica objetivo, sin inferir cumplimiento a partir del número de pruebas.

## 1. Rendimiento

| ID | Requisito | Métrica objetivo | Cómo se verifica |
|----|-----------|------------------|------------------|
| NFR-PERF-01 | Consulta de ambientes disponibles | < 300 ms (p95) | Prueba de carga sobre `GET /environments/available` |
| NFR-PERF-02 | Consulta de instructores disponibles | < 300 ms (p95) | Prueba de carga sobre `GET /instructors/available` |
| NFR-PERF-03 | Validación de horario completo (20 sesiones) | < 2 s | Prueba con horario de tamaño real |
| NFR-PERF-04 | Consulta de horario por instructor/semana | < 500 ms (p95) | Prueba de carga |
| NFR-PERF-05 | Lectura de catálogos (cacheada) | < 50 ms | Prueba contra Redis |
| NFR-PERF-06 | Publicación de evento al broker | < 100 ms | Métrica del publisher; no bloquea respuesta HTTP |

## 2. Disponibilidad

| ID | Requisito | Métrica objetivo | Ventana |
|----|-----------|------------------|---------|
| NFR-AVAIL-01 | Disponibilidad de iam-service | 99.9 % | 24/7 |
| NFR-AVAIL-02 | Disponibilidad de scheduling-service | 99.5 % | 06:00–22:00 (hora Colombia) |
| NFR-AVAIL-03 | Disponibilidad del resto de servicios | 99 % | Horas laborales |
| NFR-AVAIL-04 | Degradación parcial tolerada | scheduling opera aunque actors/environment estén caídos | Vía read models (ADR-002) |

## 3. Escalabilidad

| ID | Requisito | Métrica objetivo |
|----|-----------|------------------|
| NFR-SCALE-01 | Fichas activas concurrentes por centro | Soportar 100+ sin degradación |
| NFR-SCALE-02 | Usuarios concurrentes por centro | 200+ (coordinadores, instructores, aprendices) |
| NFR-SCALE-03 | Escalado horizontal | Cada servicio escala por réplicas sin estado compartido (JWT stateless) |
| NFR-SCALE-04 | Crecimiento a nivel nacional | Arquitectura soporta múltiples centros sin rediseño |

## 4. Seguridad

| ID | Requisito | Verificación |
|----|-----------|--------------|
| NFR-SEC-01 | Autenticación en todos los endpoints (excepto login y reset) | Revisión de contratos; pruebas de acceso sin token → 401 · **✅ Evidencia dev**: `iam` responde 401 a peticiones sin token |
| NFR-SEC-02 | Autorización por feature + scope | Pruebas de acceso cruzado entre centros → 403 SCOPE_VIOLATION · **✅ Evidencia dev**: `iam` deniega por scope/centro (403) |
| NFR-SEC-03 | Cifrado en tránsito | TLS 1.2+ en todos los canales |
| NFR-SEC-04 | Cifrado en reposo de PII y credenciales | AES-256 a nivel de BD |
| NFR-SEC-05 | Sin PII en logs | Auditoría de logs; solo IDs |
| NFR-SEC-06 | Cumplimiento Ley 1581/2012 (Habeas Data) | Ver [security-threat-model.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/05-architecture/security-threat-model.md) |
| NFR-SEC-07 | Protección contra fuerza bruta | Bloqueo de cuenta + rate limiting · **✅ Evidencia dev**: lockout 5/10 intentos verificado en `iam` |
| NFR-SEC-08 | JWT verificable localmente y rotación de llaves | Clave pública publicada vía JWKS (`/.well-known/jwks.json`); rotación de llaves de firma (DEC-008) · **✅ Evidencia dev**: JWKS servido y rotación operativa en `iam` |

## 5. Mantenibilidad

| ID | Requisito | Métrica objetivo |
|----|-----------|------------------|
| NFR-MAINT-01 | Cobertura de pruebas en servicios CORE | ≥ 80 % (scheduling, monitoring) |
| NFR-MAINT-02 | Cobertura de pruebas en servicios SUPPORTING/GENERIC | ≥ 60 % |
| NFR-MAINT-03 | Separación de capas (hexagonal) | Dominio sin imports de infraestructura (checklist de PR). La cifra histórica de 162 pruebas no es el corte actual: Service registra **186 aprobadas en CI**; las **54 IT PostgreSQL aprobadas y una omitida** corresponden a ejecución local. Los totales no prueban por sí solos separación de capas ni porcentaje de cobertura. |
| NFR-MAINT-04 | Documentación de cada servicio | README + data-model + contract + decisions |
| NFR-MAINT-05 | Naming técnico en inglés | HALT-DB-NAMING; sin términos de dominio en español en código/contratos |

## 6. Observabilidad

| ID | Requisito | Verificación |
|----|-----------|--------------|
| NFR-OBS-01 | Correlation ID en cada request | Convención transversal documentada: `X-Request-ID`. Esta entrega IAM implementa y devuelve `X-Correlation-Id`; el smoke local conserva ese encabezado. Queda por acordar/documentar su correspondencia en edge y consumidores; no se declara propagación entre todos los servicios ni un cambio del estándar global. |
| NFR-OBS-02 | Logs estructurados JSON | Campos objetivo: timestamp, level, service, request_id, event. IAM dispone de logs JSON y utiliza MDC `correlation_id`; reconciliar su correspondencia con `request_id`. Ingesta Loki y ausencia de PII del flujo completo no demostradas por este smoke. |
| NFR-OBS-03 | Métricas RED por endpoint | Rate, Errors, Duration. IAM expone contador/histograma HTTP y etiqueta `service`, comprobados localmente; faltan evidencia de scrape externo y dashboard IAM de esta entrega. |
| NFR-OBS-04 | Trazas distribuidas | Objetivo OpenTelemetry → Jaeger/Tempo. La afirmación histórica de trazas IAM visibles no dispone de anexo que acredite esta entrega: el smoke se ejecutó sin agente OTel. `X-Correlation-Id` y el fallback de errores no prueban spans ni trazas consultables; falta evidencia HTTP/JDBC y consumidor en Tempo. |
| NFR-OBS-05 | Auditoría de acciones de negocio | audit-service consume todos los eventos |
| NFR-OBS-06 | Exportación OTLP por stack | Configuración documentada: **Java** (`iam`) por **OTLP/HTTP en `:4318`** (`http/protobuf`); **Go** (`notification`) por **OTLP/gRPC en `:4317`**. Conservar la procedencia histórica por servicio. Esta revisión no acredita exportación IAM al colector ni sustituye configuración por evidencia de recepción. |
| NFR-OBS-07 | Sondas de salud diferenciadas | Split **`/health`** (liveness) vs **`/ready`** (BD y broker si está configurado). El smoke del JAR IAM devolvió 200 en ambas; no acredita readiness de toda la pila ni conectividad del broker que no se configuró para esa prueba. |

## 7. Usabilidad

| ID | Requisito | Métrica objetivo |
|----|-----------|------------------|
| NFR-USA-01 | Tiempo de creación de horario semanal | < 1 h (vs. 4–8 h manual actual) |
| NFR-USA-02 | Web responsive (desktop-first, móvil para consulta) | Funciona en pantallas ≥ 360 px |
| NFR-USA-03 | Accesibilidad | WCAG 2.1 nivel AA (objetivo) |
| NFR-USA-04 | Idioma de la interfaz | Español (audiencia SENA) |

## 8. Portabilidad / Despliegue

| ID | Requisito | Métrica objetivo |
|----|-----------|------------------|
| NFR-PORT-01 | Ambientes consistentes | DEV → QA → PROD con misma imagen (object storage compatible S3, ADR-003) |
| NFR-PORT-02 | Despliegue independiente por servicio | Cada servicio se despliega sin coordinar con otros |
| NFR-PORT-03 | Configuración por variables de entorno | Sin valores hardcodeados; secrets en Secret Manager |

## 9. Recuperación ante fallos

| ID | Requisito | Métrica objetivo |
|----|-----------|------------------|
| NFR-REC-01 | RPO (Recovery Point Objective) | ≤ 1 h (backups incrementales) |
| NFR-REC-02 | RTO (Recovery Time Objective) | ≤ 4 h por servicio |
| NFR-REC-03 | Entrega de eventos garantizada | Objetivo: at-least-once + DLQ + Outbox. El outbox IAM ya existía antes de este plan; el driver comprueba un registro staged, con stdout SQL histórico separado faltante. La IT de relay fue omitida sin broker; no se prueba publicación, entrega ni efecto en consumidor, ni rollback conjunto usuario+outbox ante fallo. DLQ/idempotencia de `notification` conservan su procedencia histórica y no acreditan el recorrido IAM de esta entrega. |
| NFR-REC-04 | Inmutabilidad de auditoría | audit_record solo INSERT; retención 7 años |

---

## Trazabilidad NFR → Arquitectura

| NFR | Decisión arquitectónica que lo soporta |
|-----|----------------------------------------|
| NFR-PERF-01/02 | Read models locales en scheduling (ADR-002) + índices optimizados |
| NFR-AVAIL-04 | Read models desacoplan scheduling de actors/environment (ADR-002) |
| NFR-SCALE-03 / NFR-SEC-08 | JWT stateless verificable localmente + JWKS y rotación de llaves (DEC-008) |
| NFR-SEC-02 | RBAC por feature+scope (rbac-design.md) |
| NFR-REC-03 | Broker con DLQ (ADR-001) + Outbox pattern |
| NFR-PORT-01 | Adaptador de storage compatible S3 (ADR-003) |
