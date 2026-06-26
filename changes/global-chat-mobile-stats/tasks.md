# Tasks: Chat Global + Stats Mobile + Notificaciones → Chat

## Fase 0 — Limpieza: chat y reacciones por partido

**API routes a eliminar:**
- [x] `src/app/api/matches/[id]/chat/route.ts`
- [x] `src/app/api/matches/[id]/chat/[msgId]/route.ts`
- [x] `src/app/api/matches/[id]/reactions/route.ts`

**Componentes a limpiar:**
- [x] `src/components/MatchInfoModal.tsx` — eliminar tab chat, estado messages/reactions/chatLoading/chatTab, funciones fetchMessages/fetchReactions/sendMessage/deleteMessage/toggleReaction, sección JSX
- [x] `src/components/MatchCard.tsx` — eliminar bloque reacciones (emoji buttons), estado reactions, fetchReactions, toggleReaction, chatActive

## Fase 1 — Eliminar buzón de notificaciones

**Archivos a eliminar:**
- [x] `src/app/notifications/page.tsx` — página historial notificaciones
- [x] `src/app/api/notifications/route.ts` — GET/POST/PUT/DELETE inbox
- [x] `src/app/api/notifications/read/route.ts` — marcar como leído

**AppContext.tsx — quitar:**
- [x] Estado `notifications`, `unreadCount`
- [x] `fetchNotifications()` y su llamada en `useEffect`
- [x] `handleMarkNotificationRead()`
- [x] Listener SSE `payload.type === 'notification'`
- [x] Exportar del context value

**AppShell.tsx — quitar:**
- [x] Import y uso de `notifications`, `unreadCount`
- [x] Campana / bell icon + badge en header (mobile y desktop)
- [x] Dropdown panel de notificaciones
- [x] Popup `show_as_popup` notifications overlay
- [x] Link "Ver historial completo" → `/notifications`

**admin/page.tsx — quitar:**
- [x] Estado `adminNotifications`, `fetchAdminNotifications`
- [x] Llamada en `useEffect`
- [x] Prop `adminNotifications` y `fetchAdminNotifications` en `<NotificationsTab>`

## Fase 2 — NotificationsTab → enviar a chat de sistema

**NotificationsTab.tsx — adaptar:**
- [x] Cambiar `fetch('/api/notifications', { method: 'POST' })` → `fetch('/api/chat/system', { method: 'POST' })`
- [x] Body: `{ message: "<titulo>: <contenido>", target_type, target_id }` (combina título + contenido en un solo mensaje)
- [x] Quitar GET de notificaciones del admin (ya no hay lista en el admin — los mensajes están en el chat)
- [x] Simplificar interfaz: solo el formulario de envío + confirmación

## Fase 3 — Base de datos

- [x] Crear `migrations/0006_global_chat.sql`:
  - Tabla `global_messages` con campos `is_system`, `target_type`, `target_id`
  - Índices en `created_at DESC` y `(target_type, target_id)`
- [x] Ejecutar migración en producción: `node scripts/migrate.js`

## Fase 4 — API Chat

- [x] Crear `src/app/api/chat/route.ts` (GET con filtro por usuario + POST mensaje normal)
- [x] Crear `src/app/api/chat/system/route.ts` (POST — admin/superadmin — is_system=TRUE, envía a Telegram también)
- [x] Crear `src/app/api/chat/[id]/route.ts` (DELETE soft — superadmin)
- [x] Agregar `/api/chat` a rate limits en `proxy.ts` (30 rps)

**Telegram en mensajes de sistema:**
- [x] En `POST /api/chat/system`: después de insertar en BD, llamar al bot Telegram con el aviso oficial (igual que hacía el sistema de notificaciones programadas)
- [x] Formato Telegram: `📣 Aviso Oficial\n<mensaje>\nDestinatario: <target>`
- [x] NO enviar Telegram por mensajes normales de usuario en el chat

## Fase 5 — ChatWidget

- [x] Crear `src/components/ChatWidget.tsx`:
  - [x] Botón flotante fixed bottom-right, badge usuarios en línea
  - [x] Modal full-screen overlay
  - [x] Lista de mensajes (scroll automático, flex-col-reverse)
  - [x] Mensajes `is_system=TRUE`: fondo ámbar, icono 📣, sin avatar de usuario
  - [x] Mensajes `is_system=FALSE`: burbuja normal, propios a la derecha
  - [x] Columna usuarios en línea (desktop) / contador (mobile)
  - [x] Input + Enviar + Enter para enviar
  - [x] Polling cada 3s con `?since=` cuando modal abierto
  - [x] Cerrar con ESC o click en overlay

## Fase 6 — AppShell: integrar ChatWidget

- [x] `src/components/AppShell.tsx` — agregar `<ChatWidget />` al final del layout (solo si user && user.aprobado)

## Fase 7 — Stats Mobile Compactas en Dashboard

- [x] `src/app/(app)/dashboard/page.tsx`:
  - [x] Agregar barra `sm:hidden` con 4 celdas compactas (Pred · Pts · Aciertos · Posición)
  - [x] Poner `hidden sm:grid` en los cards actuales

## Fase 8 — Build y Deploy

- [x] `npx tsc --noEmit` — sin errores
- [x] `NEXT_BUILD_STANDALONE=1 pnpm build`
- [x] `pm2 reload elitepass-mundial --update-env`
- [x] Bump version → v1.1.111
- [x] Commit + push GitHub
- [x] Sync openspec → GitHub → VM00
