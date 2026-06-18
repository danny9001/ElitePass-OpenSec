# elitepass-reservas: Sistema Central de Reservas y Fidelización

Núcleo operativo del ecosistema **Antigravity**. Administra eventos, reservas de mesas, venta de entradas/paquetes, control de acceso e invitados, sistema de fidelización de clientes y formularios públicos de registro.

---

## 1. Core Técnico y Arquitectura

- **Framework:** Next.js 15 (App Router) + React 19 — servidor en Node.js v24.13.0 ARM64
- **Package Manager:** pnpm 11.1.3 — **siempre `pnpm`, nunca `npm`**
- **ORM:** Prisma 6 + PostgreSQL 16
- **Cache y Colas:** Redis 7 + BullMQ (notificaciones, auditoría, puntos de fidelización)
- **Autenticación:** BetterAuth + SSO vía `elitepass-identity`
- **Estilos:** Tailwind 4 + `@theme inline` + oklch — design system acromático (fuente de verdad del ecosistema)
- **Storage:** Azure Blob Storage exclusivo — toda imagen/voucher pasa por `sharp` → WebP antes de subir
- **PM2:** modo **Cluster** × 2 workers — heap limit **1024 MB** por worker

### Arquitectura Clean Simplificada

```
src/
├── app/                    # Next.js App Router (páginas, layouts, loading/error)
├── components/             # UI Components (Radix UI + Tailwind)
├── lib/
│   ├── actions/            # Server Actions — toda la lógica de negocio
│   │   └── types/          # action-types.ts — UserRoleType, etc.
│   ├── utils/
│   │   └── role.ts         # normalizeRole() — mapeo backward-compat
│   ├── identity-client.ts  # HotSync fire-and-forget → elitepass-identity
│   └── prisma.ts           # PrismaClient singleton con PgBouncer
└── middleware.ts            # BetterAuth session guard + CSP headers
```

### Diagrama de Módulo Interno

```mermaid
graph LR
    User[Admin / Cliente] -->|HTTPS| NextJS[Next.js Server Node.js v24]

    subgraph App Router
        NextJS -->|Rutas Públicas| EventsPage[/eventos — catálogo público]
        NextJS -->|Rutas Privadas| Dashboard[/dashboard — staff]
        EventsPage -->|SSO Token| Callback[/api/auth/identity-callback]
        Dashboard -->|Server Actions| Actions[src/lib/actions]
    end

    Actions -->|Prisma + organizationId filter| DB[(jet_club_db — PgBouncer :6432)]
    Actions -->|sharp → WebP| Storage[Azure Blob Storage]
    Actions -->|job.add| Redis[(Redis BullMQ :6379/0)]
    Actions -->|fire-and-forget| Identity[elitepass-identity :3300]
    Redis -->|Worker| NotiTelegram[elitepass-noti-telegram :3200]
```

### Rendimiento

- `loading.tsx` + `error.tsx` en cada segmento de ruta — skeletons garantizan TTFB percibido < 100ms
- `html5-qrcode` y librerías de gráficas: `await import()` dinámico — solo cargan al interactuar
- `/_next/image`: proxy_cache en Nginx — evita re-procesamiento por Next Image
- `nice -n 15` en build: reduce prioridad CPU para no afectar workers en producción

---

## 2. Capa de Datos y Persistencia

Opera sobre `jet_club_db` vía PgBouncer `:6432` con `?pgbouncer=true&connection_limit=5`.

### Esquema de Entidades (ERD)

```mermaid
erDiagram
    ORGANIZATION ||--o{ EVENT : hosts
    ORGANIZATION ||--o{ USER_MEMBERSHIP : has_members
    USER ||--o{ USER_MEMBERSHIP : belongs_to
    EVENT ||--o{ REQUEST : has
    REQUEST ||--o{ TICKET : generates
    EVENT ||--o{ PUBLIC_GUEST_FORM : publishes
    PUBLIC_GUEST_FORM ||--o{ PUBLIC_GUEST_REGISTRATION : registers
    USER ||--o{ USER_ACTIVITY : records
    REQUEST ||--o{ RESERVATION_LOCK : holds

    ORGANIZATION {
        string id PK
        string name
        boolean isActive
        boolean hasCustomPaymentCredentials
        string nitPrincipal
        string paymentGatewayMode
    }
    EVENT {
        string id PK
        string organizationId FK
        string title
        datetime date
        boolean requiresInvoice
        integer publicFormMaxRegistrations
        integer publicFormInvitesPerGroup
        boolean isArchived
    }
    REQUEST {
        string id PK
        string eventId FK
        string createdById FK
        string status
        float amount
        boolean isPaid
        string paymentVoucherUrl
        string paymentQrImageUrl
        string paymentQrStatus
    }
    USER {
        string id PK
        string name
        string email
        string role
        string organizationId FK
    }
    USER_ACTIVITY {
        string id PK
        string userId FK
        datetime date
        boolean hasActivity
    }
    PUBLIC_GUEST_FORM {
        string id PK
        string eventId FK
        string createdById FK
        boolean isActive
        string slug
    }
    PUBLIC_GUEST_REGISTRATION {
        string id PK
        string formId FK
        string name
        string phone
        string status
    }
```

### Índices Críticos

- `UserActivity`: `@@unique([userId, date])`, `@@index([userId, hasActivity])` — evita seq-scan en estadísticas de fidelización
- `Request`: `@@index([eventId, status])`, `@@index([eventId, isPaid])`, `@@index([eventId, createdAt])`
- `ReservationLock`: `@@index([eventId, tableId, expiresAt])` — evita deadlocks de reservas concurrentes

### Anti-Patrones Prohibidos

- **NO** `.map()` / `.reduce()` en memoria para agregaciones — usar `groupBy`, `_count`, `_sum` de Prisma nativo
- **NO** N+1: siempre `include` / `select` apropiado o una sola query con join implícito
- **NO** `prisma.$queryRaw` sin parámetros tipados (riesgo SQL injection)

---

## 3. Mecanismos de Seguridad e Hardening

### Sistema de Roles

`UserRoleType` (canónico, 8 valores):
```ts
"SUPER_ADMIN" | "ADMIN" | "MANAGER" | "TEAM_LEADER" |
"VALIDATOR" | "COVER" | "RELACIONADOR" | "EXTERNAL"
```

`normalizeRole()` en `src/lib/utils/role.ts` — aliases backward-compat (valores legacy en DB):
```ts
USER       → RELACIONADOR   // valores antiguos de DB
SUPERVISOR → TEAM_LEADER    // valores antiguos de DB
SUPERADMIN → SUPER_ADMIN
TEAMLEADER → TEAM_LEADER
RELACIONADORA → RELACIONADOR
COVERS → COVER
```

**Lógica del callback SSO** (`/api/auth/identity-callback/route.ts`):
1. Si `JWT.role === "superadmin"` → `SUPER_ADMIN`
2. Si `JWT.empresas[].appRoles['reservas']` existe → ese rol
3. Si `JWT.accountType === "EXTERNAL" | "CLIENT"` → `EXTERNAL`
4. Fallback: rol existente en DB o `RELACIONADOR`
5. `normalizeRole()` sobre el resultado
6. Guard: si no está en `VALID_RESERVAS_ROLES` → `RELACIONADOR`
7. Prevención de degradación: usuario `EXTERNAL` no puede subir de rol por SSO

### Control de Acceso Multi-tenant

`requireOrganizationFilter()` en cada Server Action — nunca retorna datos sin filtro `organizationId`. Un usuario de la organización A nunca puede ver datos de B.

### Imágenes y Storage

- Todo archivo subido → `sharp` → WebP (PNG lossless, JPEG q95, avatar cover 200×200 q40)
- Extensiones bloqueadas: `.exe`, `.sh`, `.py`, `.php` + 20 más
- Nombre normalizado: `[YYYYMMDD]-[slug]-[uid].webp`
- Azure Blob es el único storage permitido — **nunca disco local**

### Telemetría

`measureServerAction` en todas las Server Actions críticas — registra duración, userId, resultado y errores en audit log.

### Headers de Seguridad

Middleware inyecta en todas las respuestas:
- `x-frame-options: DENY`
- `x-content-type-options: nosniff`
- `strict-transport-security: max-age=31536000; includeSubDomains`

---

## 4. Despliegue e Infraestructura

- **Puerto:** `3000` — expuesto bajo `https://reservas.genial-it.net` vía Nginx
- **Proceso PM2:** `elitepass-reservas` — modo **Cluster** × 2 — heap 1024 MB × 2
- **IDs PM2:** 3 y 4

### Comando de Deploy

```bash
cd /home/soporte/elitepass-reservas
nice -n 15 pnpm build && pm2 restart elitepass-reservas && git push danny main
```

El `nice -n 15` reduce la prioridad del build para no degradar los workers de producción durante la compilación.

### Build Notes

- `pnpm prisma generate` — siempre antes del build (evita cliente Prisma desalineado)
- `pnpm lint` (Biome) y `pnpm format` antes de commit
- Remote `danny` → `git@github.com:danny9001/sis_res.git`

### Crontab (VM00, usuario soporte)

```
15 3 * * *   curl -s http://127.0.0.1:3000/api/backups/run
0  4 * * 1   /home/soporte/mantenimiento.sh
```

### Variables de Entorno Clave

```env
DATABASE_URL="postgresql://admin:password@127.0.0.1:6432/jet_club_db?pgbouncer=true&connection_limit=5"
REDIS_URL="redis://127.0.0.1:6379/0"
IDENTITY_JWT_SECRET="secret_para_validar_sso_compartido_con_identity"
BETTER_AUTH_SECRET="secret_para_cookies_session_betterauth"
AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=...;AccountKey=..."
AZURE_STORAGE_CONTAINER_NAME="elitepass-assets"
IDENTITY_SYNC_SECRET="secreto_para_sync_identity"
PAYMENTS_API_URL="http://127.0.0.1:3100"
APP_KEY_RESERVAS="app_key_reservas_seguro"
```
