# Tasks: Chat Global + Stats Mobile + Notificaciones → Chat

## Fase 0 — Limpieza: chat y reacciones por partido

**API routes a eliminar:**
- [ ] `src/app/api/matches/[id]/chat/route.ts`
- [ ] `src/app/api/matches/[id]/chat/[msgId]/route.ts`
- [ ] `src/app/api/matches/[id]/reactions/route.ts`

**Componentes a limpiar:**
- [ ] `src/components/MatchInfoModal.tsx` — eliminar tab chat, estado messages/reactions/chatLoading/chatTab, funciones fetchMessages/fetchReactions/sendMessage/deleteMessage/toggleReaction, sección JSX
- [ ] `src/components/MatchCard.tsx` — eliminar bloque reacciones (emoji buttons), estado reactions, fetchReactions, toggleReaction, chatActive

## Fase 1 — Eliminar buzón de notificaciones

**Archivos a eliminar:**
- [ ] `src/app/notifications/page.tsx` — página historial notificaciones
- [ ] `src/app/api/notifications/route.ts` — GET/POST/PUT/DELETE inbox
- [ ] `src/app/api/notifications/read/route.ts` — marcar como leído

**AppContext.tsx — quitar:**
- [ ] Estado `notifications`, `unreadCount`
- [ ] `fetchNotifications()` y su llamada en `useEffect`
- [ ] `handleMarkNotificationRead()`
- [ ] Listener SSE `payload.type === 'notification'`
- [ ] Exportar del context value

**AppShell.tsx — quitar:**
- [ ] Import y uso de `notifications`, `unreadCount`
- [ ] Campana / bell icon + badge en header (mobile y desktop)
- [ ] Dropdown panel de notificaciones
- [ ] Popup `show_as_popup` notifications overlay
- [ ] Link "Ver historial completo" → `/notifications`

**admin/page.tsx — quitar:**
- [ ] Estado `adminNotifications`, `fetchAdminNotifications`
- [ ] Llamada en `useEffect`
- [ ] Prop `adminNotifications` y `fetchAdminNotifications` en `<NotificationsTab>`

## Fase 2 — NotificationsTab → enviar a chat de sistema

**NotificationsTab.tsx — adaptar:**
- [ ] Cambiar `fetch('/api/notifications', { method: 'POST' })` → `fetch('/api/chat/system', { method: 'POST' })`
- [ ] Body: `{ message: "<titulo>: <contenido>", target_type, target_id }` (combina título + contenido en un solo mensaje)
- [ ] Quitar GET de notificaciones del admin (ya no hay lista en el admin — los mensajes están en el chat)
- [ ] Simplificar interfaz: solo el formulario de envío + confirmación

## Fase 3 — Base de datos

- [ ] Crear `migrations/0006_global_chat.sql`:
  - Tabla `global_messages` con campos `is_system`, `target_type`, `target_id`
  - Índices en `created_at DESC` y `(target_type, target_id)`
- [ ] Ejecutar migración en producción: `node scripts/migrate.js`

## Fase 4 — API Chat

- [ ] Crear `src/app/api/chat/route.ts` (GET con filtro por usuario + POST mensaje normal)
- [ ] Crear `src/app/api/chat/system/route.ts` (POST — admin/superadmin — is_system=TRUE, envía a Telegram también)
- [ ] Crear `src/app/api/chat/[id]/route.ts` (DELETE soft — superadmin)
- [ ] Agregar `/api/chat` a rate limits en `proxy.ts` (30 rps)

**Telegram en mensajes de sistema:**
- [ ] En `POST /api/chat/system`: después de insertar en BD, llamar al bot Telegram con el aviso oficial (igual que hacía el sistema de notificaciones programadas)
- [ ] Formato Telegram: `📣 Aviso Oficial\n<mensaje>\nDestinatario: <target>`
- [ ] NO enviar Telegram por mensajes normales de usuario en el chat

## Fase 5 — ChatWidget

- [ ] Crear `src/components/ChatWidget.tsx`:
  - [ ] Botón flotante fixed bottom-right, badge usuarios en línea
  - [ ] Modal full-screen overlay
  - [ ] Lista de mensajes (scroll automático, flex-col-reverse)
  - [ ] Mensajes `is_system=TRUE`: fondo ámbar, icono 📣, sin avatar de usuario
  - [ ] Mensajes `is_system=FALSE`: burbuja normal, propios a la derecha
  - [ ] Columna usuarios en línea (desktop) / contador (mobile)
  - [ ] Input + Enviar + Enter para enviar
  - [ ] Polling cada 3s con `?since=` cuando modal abierto
  - [ ] Cerrar con ESC o click en overlay

## Fase 6 — AppShell: integrar ChatWidget

- [ ] `src/components/AppShell.tsx` — agregar `<ChatWidget />` al final del layout (solo si user && user.aprobado)

## Fase 7 — Stats Mobile Compactas en Dashboard

- [ ] `src/app/(app)/dashboard/page.tsx`:
  - [ ] Agregar barra `sm:hidden` con 4 celdas compactas (Pred · Pts · Aciertos · Posición)
  - [ ] Poner `hidden sm:grid` en los cards actuales

## Fase 8 — Build y Deploy

- [ ] `npx tsc --noEmit` — sin errores
- [ ] `NEXT_BUILD_STANDALONE=1 pnpm build`
- [ ] `pm2 reload elitepass-mundial --update-env`
- [ ] Bump version → v1.1.111
- [ ] Commit + push GitHub
- [ ] Sync openspec → GitHub → VM00
