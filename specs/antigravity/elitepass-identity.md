# elitepass-identity: Proveedor SSO e Identidad (IAM)

Proveedor de identidad (IdP) centralizado y gestor de licencias/roles del ecosistema **Antigravity**. Implementa autenticación única (SSO), acceso biométrico sin contraseña (Passkeys FIDO2) y administración multi-empresa con roles por aplicación.

---

## 1. Core Técnico y Arquitectura

- **Framework:** Next.js 15 (App Router) + Tailwind CSS 4 + Radix UI
- **Runtime:** Node.js v24.13.0 ARM64 (NVM) — heap limit 512 MB
- **Autenticación local:** better-auth v1.6 — bcrypt rounds=12 — hashes generados siempre desde Node.js (nunca shell, ya que `$` se expande al PID en bash)
- **Passkeys FIDO2:** `@simplewebauthn/server` v13.3.1 + `@simplewebauthn/browser` v13.x
  - `WEBAUTHN_RP_ID=genial-it.net`
  - `WEBAUTHN_ORIGIN=https://id.genial-it.net`
  - Challenge almacenado en tabla `WebauthnChallenge` con TTL de 5 minutos
- **Emisión de Tokens:** JWT firmados con `jsonwebtoken` (no `jose`) usando `IDENTITY_JWT_SECRET`
- **OAuth externo:** Google OAuth 2.0
- **HotSync API:** `POST /api/v1/sync` — recibe cambios de usuarios/empresas desde apps satélite vía `X-App-Secret`

### Diagrama de Flujo SSO Completo

```mermaid
graph TD
    User[Usuario / Navegador] -->|GET /login?app=&redirect=| LoginPage[Login Page]

    subgraph Métodos de Autenticación
        LoginPage -->|Email + Password| PasswordFlow[better-auth signIn.email]
        LoginPage -->|Passkey FIDO2| PasskeyFlow[startAuthentication → POST /webauthn/authenticate]
        LoginPage -->|Google OAuth| GoogleFlow[OAuth 2.0 Callback]
    end

    PasswordFlow & PasskeyFlow & GoogleFlow -->|Sesión válida| TokenRoute[GET /api/v1/token/issue]
    TokenRoute -->|buildIdentityJWT| JWT[JWT Firmado IDENTITY_JWT_SECRET]
    JWT -->|302 Redirect con ?token=| AppCallback[App Satélite /api/auth/identity-callback]
```

### Diagrama de Registro de Passkey

```mermaid
sequenceDiagram
    autonumber
    actor Usuario
    participant UI as PerfilClient.tsx
    participant Server as /api/auth/webauthn/register

    Usuario->>UI: Clic "Agregar passkey"
    UI->>Server: POST ?step=options
    Server->>Server: generateRegistrationOptions() — guarda challenge en WebauthnChallenge
    Server-->>UI: {optionsJSON}
    UI->>Browser: startRegistration({optionsJSON}) — biométrico nativo
    Browser-->>UI: {credential}
    UI->>Server: POST ?step=verify {credential}
    Server->>Server: verifyRegistrationResponse() — valida challenge
    Server->>DB: upsert passkey {credentialId, publicKey Buffer, counter}
    Server-->>UI: {success: true}
    UI->>UI: Recarga lista de passkeys
```

---

## 2. Capa de Datos y Persistencia

Opera sobre `elitepass_identity` vía PgBouncer `:6432` con `?pgbouncer=true&connection_limit=5`.

### Esquema de Entidades (ERD)

```mermaid
erDiagram
    USER ||--o{ ACCOUNT : has
    USER ||--o{ PASSKEY : registers
    USER ||--o{ WEBAUTHN_CHALLENGE : requests
    USER ||--o{ PERSONA_EMPRESA : belongs_to
    EMPRESA ||--o{ PERSONA_EMPRESA : has_members
    EMPRESA ||--o{ APP : licencias
    USER ||--o{ AUDIT_LOG : generates

    USER {
        string id PK
        string email
        string name
        string role
        string accountType
        boolean emailVerified
        datetime createdAt
    }
    ACCOUNT {
        string id PK
        string userId FK
        string providerId
        string password
        datetime updatedAt
    }
    PASSKEY {
        string id PK
        string userId FK
        string credentialId
        bytes publicKey
        integer counter
        string deviceType
        string label
        datetime createdAt
    }
    WEBAUTHN_CHALLENGE {
        string id PK
        string userId FK
        string challenge
        string type
        datetime expiresAt
    }
    EMPRESA {
        string id PK
        string nombre
        string slug
        boolean isActive
    }
    PERSONA_EMPRESA {
        string id PK
        string userId FK
        string empresaId FK
        string tipo
        json appRoles
    }
    AUDIT_LOG {
        string id PK
        string userId FK
        string targetId
        string evento
        json metadata
        datetime createdAt
    }
```

### Tabla `PERSONA_EMPRESA` — Multi-empresa

Un usuario puede pertenecer a múltiples empresas con roles distintos por aplicación:

```json
// appRoles (JSON)
{
  "pos": "CAJERO",
  "reservas": "RELACIONADOR"
}
```

Constraint único: `[userId, empresaId]` — una fila por combinación. El campo `tipo` define la categoría de membresía: `miembro | cliente | staff | admin_grupos | admin_usuarios | admin_licencias | admin_global`.

### Payload JWT emitido por `buildIdentityJWT()`

```json
{
  "sub": "user_cuid",
  "email": "usuario@genial-it.net",
  "name": "Nombre Apellido",
  "role": "admin",
  "accountType": "STAFF",
  "empresas": [
    {
      "empresaId": "empresa_cuid",
      "tipo": "admin_global",
      "appRoles": { "pos": "SUPER_ADMIN", "reservas": "ADMIN" }
    },
    {
      "empresaId": "otra_empresa_cuid",
      "tipo": "staff",
      "appRoles": { "pos": "CAJERO", "reservas": "RELACIONADOR" }
    }
  ],
  "iat": 1700000000,
  "exp": 1700043200
}
```

---

## 3. Mecanismos de Seguridad e Hardening

### Autenticación y Credenciales

- **bcrypt-12:** Estándar del ecosistema. Hash generado en Node.js con `bcrypt.hash(password, 12)`. Nunca en shell (riesgo de expansión `$var`/`$$`).
- **Super Admin canónico:** `dlandivar@genial-it.net` — contraseña estándar `L4nd1v4r$`.
- **WebAuthn:** `WEBAUTHN_RP_ID=genial-it.net` abarca todos los subdominios `*.genial-it.net`. Challenge con TTL 5 min en DB.
- **Google OAuth:** `GOOGLE_CLIENT_ID` + `GOOGLE_CLIENT_SECRET`.

### JWT y SSO

- Firmado simétricamente con `IDENTITY_JWT_SECRET` via `jsonwebtoken`.
- El mismo secreto debe estar en todas las apps satélite para verificar la firma.
- Expiración configurable (default: 12 horas en callback).

### Mapeo de Roles hacia Apps Satélite

El callback de cada app lee el JWT en este orden de prioridad:

1. Si `JWT.role === "superadmin"` → rol destino = `SUPER_ADMIN`
2. Si `JWT.empresas[].appRoles['reservas'|'pos']` existe → usar ese valor
3. Si `JWT.accountType === "EXTERNAL" | "CLIENT"` → `EXTERNAL`
4. Fallback al rol existente en DB local o `RELACIONADOR`
5. `normalizeRole()`: `USER→RELACIONADOR`, `SUPERVISOR→TEAM_LEADER` (aliases backward-compat)
6. Guard: si el rol final no es válido → `RELACIONADOR`

### Audit Log

Todas las acciones de administración (asignación/remoción de membresías, cambios de rol, login fallido) se registran en `AuditLog` con `evento`, `userId`, `targetId`, `metadata JSON`.

### HotSync Security

El endpoint `POST /api/v1/sync` exige el header `X-App-Secret` que debe coincidir con `X_APP_SECRET`. Si falla → HTTP 401. Todas las operaciones internas son idempotentes (upsert).

---

## 4. Despliegue e Infraestructura

- **Puerto:** `3300` — expuesto bajo `https://id.genial-it.net` vía Nginx
- **Proceso PM2:** `elitepass-identity` — modo **Fork** — heap 512 MB
- **Build:** `pnpm build` (Next.js standalone output)
- **Deploy:**
  ```bash
  cd /home/soporte/elitepass-identity
  pnpm build && pm2 restart elitepass-identity
  ```

### Rutas Principales

| Ruta | Descripción |
|---|---|
| `GET /login` | Login SSO (email/password + passkey) |
| `GET /api/v1/token/issue` | Emite JWT y redirige al callback |
| `POST /api/v1/sync` | HotSync desde apps satélite |
| `POST /api/auth/webauthn/register?step=options|verify` | Registro de passkey |
| `POST /api/auth/webauthn/authenticate` | Login con passkey |
| `GET /api/auth/webauthn/passkeys` | Lista passkeys del usuario autenticado |
| `DELETE /api/auth/webauthn/passkeys?id=` | Elimina passkey |
| `GET /dashboard/personas` | Admin: lista de personas (filtrable por empresa) |
| `GET /dashboard/personas/[id]` | Admin: detalle con `EmpresaMembershipsManager` |
| `GET /dashboard/empresas` | Admin: lista de empresas registradas |
| `GET /dashboard/empresas/[id]` | Admin: detalle de empresa, licencias y miembros |
| `GET /dashboard/empresas/[id]/editar` | Admin: formulario dedicado de edición (Datos Generales + Asignación de miembros con buscador) |


### Variables de Entorno

```env
BETTER_AUTH_URL="https://id.genial-it.net"
BETTER_AUTH_SECRET="secret_de_inicializacion_better_auth"
DATABASE_URL="postgresql://user:password@127.0.0.1:6432/elitepass_identity?pgbouncer=true&connection_limit=5"
IDENTITY_JWT_SECRET="jwt_secret_firmado_sso_compartido"
X_APP_SECRET="secreto_para_validar_peticiones_sync"
WEBAUTHN_RP_ID="genial-it.net"
WEBAUTHN_ORIGIN="https://id.genial-it.net"
GOOGLE_CLIENT_ID="google_oauth_client_id"
GOOGLE_CLIENT_SECRET="google_oauth_client_secret"
```
