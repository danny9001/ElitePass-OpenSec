# elitepass-pos: ERP y Punto de Venta (POS)

Gestión de inventario de bebidas/insumos, barras físicas en locales nocturnos, traspasos, devoluciones y flujo de ventas con cobros en efectivo, tarjeta o QR. Incluye branding personalizable por tenant.

---

## 1. Core Técnico y Arquitectura

### Estructura Cliente-Servidor Desacoplada

| Componente | Tecnología | Notas |
|---|---|---|
| **Frontend** | Vite 6 + React 19 + TypeScript | SPA compilada, servida por nginx desde `dist/` |
| **Backend** | Node.js v24.13.0 + Express 5 + TypeScript | Ejecutado con `tsx` — NO se compila, corre fuente directamente |
| **ORM** | Prisma 6 + PostgreSQL 16 | `elite_pass_db` vía PgBouncer :6432 |
| **Store** | Zustand | Estado global de sesión, caja, stock |
| **WebSockets** | Socket.IO | Actualizaciones de stock y pedidos en tiempo real a barras |
| **Estilos** | Tailwind CSS 3 + CSS Variables `--surface-*` | Tema claro (light) por defecto — system fonts |
| **Login** | `public/login.html` estático | Servido directamente por nginx (`location = /login`) — **NO es React** |

### Arquitectura de Login Estático

El archivo `frontend/public/login.html` es un HTML auto-contenido con CSS inline y JS vanilla. Nginx intercepta `/login` antes de que llegue al servidor Node.js, lo que elimina el cold-start de React y garantiza TTFB < 100ms. El formulario hace `POST /api/v1/auth/login` (Express), recibe JWT y redirige al SPA.

### Diagrama de Flujo POS

```mermaid
graph TD
    LoginHTML[login.html — nginx static] -->|POST /api/v1/auth/login| Express[Express API :3001]
    Express -->|JWT firmado| SPA[React SPA — Vite build]

    SPA -->|Venta / Pedido| Express
    SPA -->|WebSocket| WS[Socket.IO Server]
    WS -->|Broadcast stock| Bars[Terminales Bartenders]

    Express -->|Prisma| DB[(elite_pass_db — PgBouncer :6432)]
    Express -->|X-App-Key| Payments[elitepass-payments :3100]
    Express -->|fire-and-forget| IdentitySync[elitepass-identity /api/v1/sync]
    Express -->|x-api-key| NotiTelegram[elitepass-noti-telegram :3200]
```

### Resiliencia del Frontend

- `ChunkErrorBoundary` en `main.tsx`: detecta `ChunkLoadError` (Service Worker con cache obsoleta) y ejecuta recarga automática única — evita pantalla en blanco tras deploy.
- `PWA Service Worker` (`sw.js`): caché de assets estáticos con estrategia network-first para la API.

---

## 2. Capa de Datos y Persistencia

Opera sobre `elite_pass_db` vía PgBouncer `:6432`. Toda consulta lleva filtro `empresaId` implícito.

### Esquema de Entidades (ERD)

```mermaid
erDiagram
    EMPRESA ||--o{ EVENTO : organiza
    EMPRESA ||--o{ USUARIO : tiene
    EVENTO ||--o{ CAJA_SESION : tiene
    CAJA_SESION ||--o{ VENTA : registra
    VENTA ||--o{ VENTA_DETALLE : contiene
    PRODUCTO ||--o{ VENTA_DETALLE : vendido_en
    BARRA ||--o{ PRODUCTO_BARRA : tiene_stock
    BARRA ||--o{ TRASPASO : origen_o_destino
    PRODUCTO ||--o{ TRASPASO_DETALLE : incluido_en

    EMPRESA {
        string id PK
        string nombre
        string slug
        string logoUrl
        string brandName
        string primaryColor
    }
    USUARIO {
        string id PK
        string empresaId FK
        string email
        string role
        string nombre
    }
    PRODUCTO {
        string id PK
        string empresaId FK
        string nombre
        float precioVenta
        integer stockActual
        string unidad
    }
    VENTA {
        string id PK
        string cajaSesionId FK
        float total
        string metodoPago
        string estado
        datetime createdAt
    }
    CAJA_SESION {
        string id PK
        string usuarioId FK
        string eventoId FK
        float fondoApertura
        float fondoCierre
        datetime fechaApertura
        datetime fechaCierre
    }
    TRASPASO {
        string id PK
        string barraOrigenId FK
        string barraDestinoId FK
        string solicitanteId FK
        string estado
        datetime createdAt
    }
```

### Índices Críticos

- `Venta`: `@@index([cajaSesionId, estado])`, `@@index([createdAt])`
- `ProductoBarra`: `@@unique([productoId, barraId])` — stock por barra
- `CajaSesion`: `@@index([eventoId, usuarioId, fechaCierre])` — arqueo de caja

---

## 3. Mecanismos de Seguridad e Hardening

### JWT Local del POS

El backend Express emite su propio JWT (`JWT_SECRET`) al autenticar en `/api/v1/auth/login`. Este JWT es diferente del JWT SSO de Identity — es para sesiones internas del POS.

- Access token: 8h (configurable)
- Refresh token: 7 días (`JWT_REFRESH_SECRET`)
- Payload: `{ userId, empresaId, role, nombre }`

### Roles y Permisos

| Rol | Abrir/Cerrar Caja | Vender | Traspasos | Compras/Almacén | Reportes | Branding |
|---|---|---|---|---|---|---|
| **SUPER_ADMIN** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **ADMIN_SISTEMA** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **ADMIN_ALMACEN** | ❌ | ❌ | ✅ Aprobar | ✅ | ❌ | ❌ |
| **CAJERO** | ✅ Propia | ✅ | ✅ Solicitar | ❌ | ❌ | ❌ |
| **BARTENDER** | ❌ | ✅ Barra propia | ✅ Confirmar | ❌ | ❌ | ❌ |
| **CONTADOR** | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |

### Seguridad de Archivos e Imágenes

- Imágenes subidas convertidas a WebP via `sharp` antes de subir a Azure Blob Storage
- Extensiones bloqueadas: `.exe`, `.sh`, `.py`, `.php` + 20 más
- Branding (logos, fondos) almacenados en Azure Blob — nunca en disco local

### Sincronización con Identity

`backend/src/config/identity-sync.ts` — fire-and-forget al crear usuarios o empresas:
```ts
POST /api/v1/sync  →  X-App-Secret: IDENTITY_SYNC_SECRET
```

---

## 4. Despliegue e Infraestructura

- **Puerto:** `3001` — expuesto bajo `https://pos.genial-it.net` vía Nginx
- **Proceso PM2:** `elitepass-pos` — modo **Fork** — heap limit **384 MB**
- **Backend:** `tsx src/server.ts` — sin paso de build (fuente TypeScript directamente)
- **Frontend:** Build Vite, assets en `frontend/dist/` servidos por Nginx

### Build del Frontend (POS)

```bash
cd /home/soporte/elitepass-pos/frontend
# NOTA: pnpm build puede fallar por ERR_PNPM_IGNORED_BUILDS:esbuild
# Alternativa directa:
node_modules/.bin/vite build
```

### Rutas Nginx (pos.genial-it.net)

```nginx
location = /login {
    root /home/soporte/elitepass-pos/frontend/public;
    try_files /login.html =404;
}
location /api/ {
    proxy_pass http://127.0.0.1:3001;
}
location / {
    root /home/soporte/elitepass-pos/frontend/dist;
    try_files $uri /index.html;
}
```

### Design System (Estándar Ecosistema)

- **Fuentes:** `"Segoe UI", "Helvetica Neue", Helvetica, Arial, system-ui` — prohibido Google Fonts
- **Tema:** Claro (light) por defecto — `--surface: #f8fafc`, cards en `#ffffff`
- **Border radius:** `--radius: 0.625rem` (10px)
- **Colores de estado:** Pastel sobre fondo claro (success `#f0fdf4`, warning `#fffbeb`, danger `#fef2f2`)
- **Source of truth:** `elitepass-reservas` globals.css — POS replica la paleta via variables `--surface-*`

### Variables de Entorno

```env
DATABASE_URL="postgresql://elitepass:password@127.0.0.1:6432/elite_pass_db?pgbouncer=true&connection_limit=5"
REDIS_URL="redis://127.0.0.1:6379/1"
JWT_SECRET="jwt_secreto_pos_auth"
JWT_REFRESH_SECRET="jwt_secreto_refresh_token"
VITE_API_URL="https://pos.genial-it.net/api/v1"
IDENTITY_SYNC_SECRET="secreto_para_sync_identity"
AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;..."
VAPID_PUBLIC_KEY="clave_publica_push_notifications"
VAPID_PRIVATE_KEY="clave_privada_push_notifications"
```
