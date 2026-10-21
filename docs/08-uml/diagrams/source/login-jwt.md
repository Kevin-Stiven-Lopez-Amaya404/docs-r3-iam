# Secuencia — Login + emisión de JWT (`iam-service`)

> Estado: 🟡 Revisión | Última actualización: 2026-10-21
> Autor: Jesús Ariel González Bonilla (PM + Arquitecto) | Equipo: Arquitectura
> Alcance: autenticación existente y ampliación R3 en PR abiertas; smoke HTTP local del adaptador, sin montaje del web-shell en navegador. Fuentes: [contrato IAM](../../../09-microservices/services/01-iam-service/components/iam-api/contract.md), [decisiones](../../../09-microservices/services/01-iam-service/decisions.md) y [modelo de datos](../../../09-microservices/services/01-iam-service/data-model.md).

El login existente valida credenciales, lockout y RBAC, emite un access token
JWT RS256 y un refresh token, y persiste sesión, auditoría y staging outbox.
La entrega nueva corrige el consumidor MFE y añade correlación/métricas al
servicio; no crea esas capacidades de negocio. Los TTL de referencia del contrato
son 15 minutos para access y siete días para refresh.

Leyenda: el recorrido driver → adaptador → proxy → API → PostgreSQL fue probado
localmente. Los detalles de negocio se respaldan por regresiones del servicio.
Host, callback en el shell, consumidor de eventos y trazas distribuidas están
pendientes de validación integrada; no forman parte del recorrido local probado.

```mermaid
sequenceDiagram
    autonumber
    actor Driver as Driver HTTP local
    participant MFE as Adaptador MFE
    participant Proxy as Proxy Vite (15101)
    participant Filter as CorrelationIdFilter
    participant API as iam-api (AuthController)
    participant Svc as AuthenticateUser (existente)
    participant DB as PostgreSQL aislado
    participant Host as Host (contrato pendiente)

    Driver->>MFE: authenticate(email, password)
    MFE->>Proxy: POST /api/v1/auth/login + X-Correlation-Id
    Proxy->>Filter: Petición al JAR (18080)
    Filter->>Filter: Validar/generar ID y establecer MDC
    Filter->>API: Petición correlacionada
    API->>Svc: authenticate(email, password)
    Svc->>DB: SELECT user WHERE email
    DB-->>Svc: user (hash bcrypt, failed_attempts, locked_until)

    alt Usuario bloqueado (locked_until > now)
        Svc-->>API: LockedException
        API-->>MFE: 401 + envelope error + X-Correlation-Id (vía proxy)
    else Credenciales inválidas
        Svc->>Svc: bcrypt.verify(password, hash) = false
        Svc->>DB: UPDATE failed_attempts += 1 (lockout si excede umbral)
        Svc->>DB: INSERT audit_login (outcome = INVALID_PASSWORD)
        Svc-->>API: BadCredentialsException
        API-->>MFE: 401 + envelope error + X-Correlation-Id (vía proxy)
    else Credenciales válidas
        Svc->>Svc: bcrypt.verify(password, hash) = true
        Svc->>DB: reset failed_attempts = 0
        Svc->>Svc: cargar roles/features/scopes (RBAC)
        Svc->>Svc: Emitir access JWT RS256 y refresh token
        Note over Svc,DB: misma transacción
        Svc->>DB: INSERT refresh_token (hash SHA-256)
        Svc->>DB: INSERT audit_login (outcome = SUCCESS)
        Svc->>DB: INSERT identity.outbox (payload JSONB contiene envelope.data)
        Svc-->>API: TokenPair (access, refresh)
        API-->>MFE: 200 TokenResponse plano + X-Correlation-Id (vía proxy)
        MFE-->>Driver: Sesión adaptada y validada
    end
    Filter->>Filter: Restaurar MDC anterior (también ante excepción)
    opt Contrato del host; no ejecutado por el smoke local
        MFE-->>Host: onAuthenticated(session)
        Note over MFE,Host: El host puede inyectar AuthenticationPort/fetch instrumentado
        Note over MFE,Host: Montaje en navegador y span W3C real pendientes
    end
```

## Notas

- El bloque `INSERT identity.outbox` deja el evento en *staging*; su publicación a `iam-events`
  la realiza el relay de forma asíncrona — ver [outbox-relay-events.md](https://github.com/code-sena/design-software-docs/blob/917d7f6b514a4ff48d86f7ae78273f06e0df8a1c/08-uml/diagrams/source/outbox-relay-events.md).
- El envelope se persiste en la columna JSONB `identity.outbox.payload`; el campo JSON `data` contiene los datos de negocio. El relay lo transforma al envelope de publicación con campo `payload`. No hay una columna SQL `data` en outbox.
- La clave pública de verificación RS256 se expone vía JWKS ([DEC-008](../../../09-microservices/services/01-iam-service/decisions.md)); esa capacidad es previa al plan.
- `TokenResponse` contiene `access_token`, `refresh_token`, `token_type`,
  `expires_in` y `user`; no hay wrapper `data`. El callback es el contrato
  esperado por la pantalla: el smoke no acredita navegación, cookies o ciclo
  de sesión del shell.
- Registros históricos: 186 casos de servicio y 54 IT PostgreSQL aprobadas,
  una IT broker omitida; 46 casos MFE. La CI de Service omite las 55 IT externas;
  la del MFE no llegó a probar por falta de acceso a dependencias privadas.
- El filtro restaura MDC y el endpoint Prometheus expone contador/histograma;
  el smoke JAR no cargó agente OTel. No hay evidencia en esta entrega de spans
  HTTP/JDBC en Tempo, scrape, Loki, alerta o publicación/consumo RabbitMQ.
- Service [#43](https://github.com/code-sena/design-software-iam-service/pull/43),
  DB [#79](https://github.com/code-sena/design-software-iam-db/pull/79) y MFE
  [#10](https://github.com/code-sena/design-software-iam-mfe/pull/10) estaban
  abiertas y sin fusionar al corte. La
  [matriz de evidencias](../../../11-quality/plan-mejoramiento/iam-kevin-stiven-lopez-amaya/matriz-evidencias.md)
  fija versiones y resultados. No se reejecutaron pruebas para este diagrama.
