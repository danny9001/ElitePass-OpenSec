# Design: Chat Global en Vivo + Stats Compactas Mobile + Notificaciones → Chat

---

## Feature 1: Chat Global (reemplaza buzón de notificaciones)

### Decisión central

El buzón de notificaciones (campana, `/notifications`, `unreadCount`) se **elimina completamente**. Las notificaciones que los admins envíen pasan a ser **mensajes de sistema en el chat**, visibles solo para los destinatarios correctos (todos / empresa / grupo / usuario).

Un solo canal de comunicación: el chat global flotante.

---

### Por qué PostgreSQL polling y no SSE/Redis

El cluster PM2 tiene 4 procesos independientes. `realtimeEmitter` es un `EventEmitter` en memoria — local a cada proceso. Un mensaje del proceso 1 no llega a clientes en proceso 2/3/4.

| Opción | Complejidad | Cluster-safe |
|---|---|---|
| SSE + global emitter | Media | ❌ |
| Redis pub/sub | Alta | ✅ requiere Redis |
| **PostgreSQL polling** | **Baja** | **✅** |

**Decisión: polling cada 3s con `?since=<ISO>`** — simple, sin dependencias nuevas.

---

### Base de datos — `global_messages`

```sql
-- migrations/0006_global_chat.sql
CREATE TABLE IF NOT EXISTS global_messages (
    id            SERIAL PRIMARY KEY,
    user_id       INTEGER REFERENCES users(id) ON DELETE SET NULL,
    message       TEXT NOT NULL CHECK (char_length(message) BETWEEN 1 AND 500),
    is_system     BOOLEAN NOT NULL DEFAULT FALSE,   -- TRUE = notificación de admin
    target_type   VARCHAR(20) NOT NULL DEFAULT 'all', -- 'all'|'user'|'company'|'group'
    target_id     INTEGER,                          -- NULL cuando target_type='all'
    created_at    TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    deleted_at    TIMESTAMP WITH TIME ZONE,
    deleted_by_id INTEGER REFERENCES users(id)
);
CREATE INDEX IF NOT EXISTS idx_global_messages_created ON global_messages (created_at DESC);
CREATE INDEX IF NOT EXISTS idx_global_messages_target  ON global_messages (target_type, target_id);
```

Las tablas `notifications` y `notification_reads` se **mantienen en BD** (datos históricos).  
La tabla `push_subscriptions` y el mecanismo de Web Push se **mantienen** (push a dispositivo es distinto al inbox).

---

### API `/api/chat`

#### `GET /api/chat`
- Auth: usuario aprobado
- Devuelve mensajes visibles para el usuario actual:
  ```sql
  WHERE deleted_at IS NULL
    AND (
      is_system = FALSE                              -- mensaje de usuario (todos lo ven)
      OR target_type = 'all'                         -- aviso para todos
      OR (target_type = 'user'    AND target_id = $user_id)
      OR (target_type = 'company' AND target_id IN (SELECT company_id FROM user_companies WHERE user_id=$user_id))
      OR (target_type = 'group'   AND target_id IN (SELECT group_id  FROM user_groups    WHERE user_id=$user_id))
    )
  ```
- Query params: `?since=<ISO>` → solo mensajes nuevos | sin param → últimos 80
- JOIN con `users` para nombre + avatar + tipo

#### `POST /api/chat` — mensaje de usuario
- Auth: usuario aprobado (cualquier rol)
- Body: `{ message: string }` (1–500 chars)
- `is_system = FALSE`, `target_type = 'all'`
- Rate limit: verificar que el user no envió un mensaje en el último segundo
- Sanitiza con `sanitizeText()`

#### `POST /api/chat/system` — aviso oficial (reemplaza POST /api/notifications)
- Auth: admin o superadmin
- Body: `{ message, target_type, target_id? }` — misma lógica de targeting que tenía `/api/notifications`
- `is_system = TRUE`, `user_id = admin.id`
- Guarda en `global_messages` en lugar de `notifications`
- Llama `broadcastUpdate('chat', ...)` para avisar a clientes con SSE abierto
- `logSystem(...)` igual que antes

#### `DELETE /api/chat/[id]`
- Auth: superadmin únicamente
- Soft delete: `deleted_at = NOW(), deleted_by_id = user.id`

---

### Renderizado en el chat — mensajes de sistema vs usuario

```
┌─────────────────────────────────────────────────────────┐
│  📣  Aviso Oficial                       09:32 AM       │  ← is_system=TRUE
│  Recuerda que los pronósticos cierran 15 min antes.     │
│  Destinatario: Empresa Acme                             │
└─────────────────────────────────────────────────────────┘

│  Avatar  Juan Pérez               09:35 AM  │            ← is_system=FALSE
│          ¡Vamos Bolivia! 🇧🇴                │
```

- `is_system = TRUE` → fondo ámbar/naranja, icono 📣, sin foto de perfil, muestra quién lo envió
- `is_system = FALSE` → burbuja normal, propios a la derecha

---

### Componente `ChatWidget.tsx`

```
src/components/ChatWidget.tsx   ← 'use client'
```

**Botón flotante:**
```
fixed bottom-6 right-6 z-40
rounded-full w-14 h-14 bg-yellow-500 shadow-xl
badge verde con count usuarios en línea
```

**Modal full-screen:**
```
┌────────────────────────────────────────────────────┐
│ 💬 Chat del Torneo              🟢 N en línea  [X] │
├──────────────────────────┬─────────────────────────┤
│                          │  EN LÍNEA (desktop)     │
│   scroll mensajes        │  • Avatar Nombre        │
│   (flex-col-reverse)     │  • Avatar Nombre        │
│   📣 avisos en ámbar     │                         │
├──────────────────────────┴─────────────────────────┤
│ [Av]  [Escribe un mensaje...............] [Enviar] │
└────────────────────────────────────────────────────┘
```

- Mobile: sin columna lateral, solo contador en header
- Scroll automático al recibir mensajes nuevos
- Cerrar con ESC o click en overlay

**Integración:** `ChatWidget` se agrega en `AppShell.tsx` (una vez, todas las páginas).

---

### Lo que se elimina del sistema de notificaciones

| Elemento | Acción |
|---|---|
| Campana + badge `unreadCount` en AppShell | Eliminar |
| Dropdown de notificaciones en AppShell | Eliminar |
| Página `/notifications` | Eliminar |
| `notifications`, `unreadCount` en AppContext | Eliminar |
| `GET /api/notifications` (inbox usuario) | Eliminar |
| `POST /api/notifications/read` | Eliminar |
| `show_as_popup` popup en AppShell | Eliminar |
| `NotificationsTab` en admin | Adaptar → usa `POST /api/chat/system` |

| Elemento | Se mantiene |
|---|---|
| Web Push (`push_subscriptions`, `/api/push`) | ✅ |
| **Telegram bot — solo avisos oficiales** | ✅ |
| Tablas `notifications`, `notification_reads` en BD | ✅ histórico |
| `logSystem` audit trail | ✅ |
| `notify-scheduled` cron | ✅ (push/telegram) |

**Regla Telegram:** el bot solo envía mensajes cuando un **admin/superadmin publica un aviso oficial** (`is_system=TRUE`) desde el panel. Los mensajes normales de usuarios en el chat NO se reenvían a Telegram.

---

## Feature 2: Stats Compactas Mobile

En `dashboard/page.tsx`, dos vistas:

```tsx
{/* Mobile: barra compacta — sm:hidden */}
<div className="sm:hidden grid grid-cols-4 gap-1.5 ...">
  {/* Valor grande + label tiny, sin sub-text */}
</div>

{/* Desktop: cards actuales — hidden sm:grid */}
<div className="hidden sm:grid grid-cols-2 xl:grid-cols-4 ...">
  {/* Sin cambios */}
</div>
```

Altura mobile: ~72px total vs ~260px actual.

---

## Archivos a crear / modificar / eliminar

| Archivo | Acción |
|---|---|
| `migrations/0006_global_chat.sql` | CREAR |
| `src/app/api/chat/route.ts` | CREAR (GET + POST usuario) |
| `src/app/api/chat/system/route.ts` | CREAR (POST sistema — reemplaza notifications POST) |
| `src/app/api/chat/[id]/route.ts` | CREAR (DELETE superadmin) |
| `src/components/ChatWidget.tsx` | CREAR |
| `src/components/AppShell.tsx` | MODIFICAR: quitar campana/badge/dropdown/popup; agregar `<ChatWidget />` |
| `src/contexts/AppContext.tsx` | MODIFICAR: quitar `notifications`, `unreadCount`, fetch notifs, markRead |
| `src/components/admin/NotificationsTab.tsx` | MODIFICAR: POST a `/api/chat/system` en lugar de `/api/notifications` |
| `src/app/(app)/admin/page.tsx` | MODIFICAR: quitar `adminNotifications`, `fetchAdminNotifications` |
| `src/app/(app)/dashboard/page.tsx` | MODIFICAR: stats compactas mobile |
| `src/app/notifications/page.tsx` | ELIMINAR |
| `src/app/api/notifications/route.ts` | ELIMINAR (GET/PUT/DELETE inbox) |
| `src/app/api/notifications/read/route.ts` | ELIMINAR |
| `src/proxy.ts` | MODIFICAR: agregar `/api/chat` rate limit 30 rps |
| `src/app/api/matches/[id]/chat/` | ELIMINAR (carpeta completa) |
| `src/app/api/matches/[id]/reactions/` | ELIMINAR (carpeta completa) |
| `src/components/MatchInfoModal.tsx` | MODIFICAR: quitar tab chat + reacciones |
| `src/components/MatchCard.tsx` | MODIFICAR: quitar reacciones |
