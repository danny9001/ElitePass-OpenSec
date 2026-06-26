# Design: Chat Global en Vivo + Stats Compactas Mobile

---

## Feature 1: Chat Global

### Arquitectura — Por qué PostgreSQL polling y no SSE/Redis

El cluster PM2 tiene 4 procesos independientes. `realtimeEmitter` es un `EventEmitter` en memoria — local a cada proceso. Un mensaje enviado al proceso 1 no llega a clientes conectados al proceso 2, 3 o 4.

Opciones evaluadas:

| Opción | Complejidad | Cluster-safe | Requiere |
|---|---|---|---|
| SSE + global emitter | Media | ❌ Solo mismo proceso | Nada extra |
| Redis pub/sub | Alta | ✅ | Redis configurado (existe en VM02 pero no conectado) |
| **PostgreSQL polling** | **Baja** | **✅** | **Nada extra** |
| PG LISTEN/NOTIFY | Media | ✅ | Una conexión PG permanente por proceso |

**Decisión: PostgreSQL polling cada 3 segundos** — el cliente envía `?since=<ISO>` y la API devuelve solo mensajes nuevos. Simple, funciona en cluster, la BD ya está ahí.

---

### Base de datos

```sql
-- migrations/0006_global_chat.sql
CREATE TABLE IF NOT EXISTS global_messages (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    message     TEXT    NOT NULL CHECK (char_length(message) BETWEEN 1 AND 500),
    created_at  TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    deleted_at  TIMESTAMP WITH TIME ZONE,
    deleted_by_id INTEGER REFERENCES users(id)
);
CREATE INDEX IF NOT EXISTS idx_global_messages_created ON global_messages (created_at DESC);
```

---

### API Routes

#### `GET /api/chat`
- Auth: `getSessionUser()`, debe ser aprobado
- Query: `?since=<ISO>` (opcional) — devuelve mensajes posteriores a esa fecha
- Sin `since`: últimos 80 mensajes
- Response: `[{ id, user_id, nombre, avatar, message, created_at }]`
- JOIN con `users` para nombre + avatar

#### `POST /api/chat`
- Auth: `getSessionUser()`, debe ser aprobado
- Body: `{ message: string }` (1–500 chars)
- Sanitiza con `sanitizeText()`
- Rate limit: máx 1 mensaje/segundo (verificar `created_at` del último mensaje del user)
- Response: `{ id, created_at }`

#### `DELETE /api/chat/[id]`
- Auth: superadmin únicamente
- Soft delete: `deleted_at = NOW(), deleted_by_id = user.id`

---

### Componente `ChatWidget.tsx`

```
/src/components/ChatWidget.tsx   ← cliente (use client)
```

**Estado:**
- `open: boolean` — modal abierto/cerrado
- `messages: Message[]` — mensajes cargados
- `onlineUsers: User[]` — usuarios en línea
- `input: string` — texto del mensaje
- `sending: boolean`
- `lastSince: string` — timestamp del último mensaje recibido

**Ciclo de vida:**
1. Mount → `fetchMessages()` (últimos 80)
2. `setInterval(3000)` → `fetchNewMessages(?since=lastSince)` mientras `open === true`
3. Cerrar modal → clearInterval

**Layout del modal (full-screen overlay):**
```
┌─────────────────────────────────────────────────┐
│ [X]  Chat del Torneo            [N en línea 🟢] │
├────────────────────────┬────────────────────────┤
│                        │  ONLINE (desktop only) │
│   Mensajes scroll      │  • Avatar Nombre       │
│   (flex-col-reverse)   │  • Avatar Nombre       │
│                        │  • ...                 │
├────────────────────────┴────────────────────────┤
│ [Avatar] [Input texto.....................] [▶] │
└─────────────────────────────────────────────────┘
```
- Mobile: sin columna lateral de usuarios en línea (solo header con contador)
- Desktop: columna derecha fija 200px con lista de usuarios en línea
- Mensajes propios: alineados a la derecha, color diferente
- Scroll automático al último mensaje al abrir y al recibir nuevos

**Botón flotante:**
```
fixed bottom-6 right-6 z-40
rounded-full w-14 h-14 bg-yellow-500
badge rojo con count de usuarios en línea
```

**Integración en layout:** Se agrega en `AppShell.tsx` una sola vez, fuera del contenido de página, para que aparezca en todas las rutas.

---

### Rate limiting en API de chat

En `proxy.ts` ya existe el patrón. Se añade:
```ts
{ pattern: /^\/api\/chat/, rps: 30, windowMs: 60_000 },
```
Protección adicional en el route handler: verificar que el mismo user no envió un mensaje en el último segundo.

---

## Feature 2: Stats Compactas Mobile

### Problema
En mobile, el grid `grid-cols-2` genera 4 cards de ~130px de alto cada uno = 260px de scroll antes de llegar al contenido principal.

### Solución
Dos vistas del mismo dato:

```tsx
{/* Mobile: barra compacta — sm:hidden */}
<div className="sm:hidden grid grid-cols-4 gap-2 ...">
  { [Pred, Pts, Aciertos, Posición] }  // valor grande + label 2 líneas tiny
</div>

{/* Desktop: cards actuales — hidden sm:grid */}
<div className="hidden sm:grid grid-cols-2 xl:grid-cols-4 ...">
  { /* sin cambios */ }
</div>
```

La barra mobile muestra solo el número y el label abreviado en cada celda, sin `sub` text. Altura total ≈ 72px en lugar de 260px.

---

## Archivos a crear/modificar

| Archivo | Acción |
|---|---|
| `migrations/0006_global_chat.sql` | CREAR |
| `src/app/api/chat/route.ts` | CREAR (GET + POST) |
| `src/app/api/chat/[id]/route.ts` | CREAR (DELETE superadmin) |
| `src/components/ChatWidget.tsx` | CREAR |
| `src/components/AppShell.tsx` | MODIFICAR — agregar `<ChatWidget />` |
| `src/app/(app)/dashboard/page.tsx` | MODIFICAR — stats mobile compactas |
| `src/proxy.ts` | MODIFICAR — rate limit `/api/chat` |
