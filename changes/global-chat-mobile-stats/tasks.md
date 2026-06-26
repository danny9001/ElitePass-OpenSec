# Tasks: Chat Global en Vivo + Stats Compactas Mobile

## Fase 0 — Limpieza: eliminar chat y reacciones por partido

**API routes a eliminar (carpetas completas):**
- [ ] `src/app/api/matches/[id]/chat/route.ts`
- [ ] `src/app/api/matches/[id]/chat/[msgId]/route.ts`
- [ ] `src/app/api/matches/[id]/reactions/route.ts`

**Componentes a limpiar:**
- [ ] `src/components/MatchInfoModal.tsx` — eliminar tab "chat", estado messages/reactions/chatLoading/chatTab, fetchMessages(), fetchReactions(), sendMessage(), deleteMessage(), toggleReaction(), sección JSX del chat y reacciones
- [ ] `src/components/MatchCard.tsx` — eliminar bloque de reacciones (emoji buttons), estado reactions, fetchReactions(), toggleReaction(), `chatActive` check

**Mantener en BD** (datos históricos, no borrar tablas):
- `match_messages` — conservar
- `match_reactions` — conservar
- `is_moderador` en `users` — conservar (puede usarse en chat global)

## Fase 1 — Base de datos
- [ ] Crear `migrations/0006_global_chat.sql` con tabla `global_messages`
- [ ] Ejecutar migración en producción: `node scripts/migrate.js`
- [ ] Verificar índice en `created_at DESC`

## Fase 2 — API
- [ ] Crear `src/app/api/chat/route.ts` (GET + POST)
- [ ] Crear `src/app/api/chat/[id]/route.ts` (DELETE soft — superadmin)
- [ ] Agregar `/api/chat` a rate limits en `proxy.ts` (30 rps)
- [ ] Test: POST un mensaje, GET devuelve con JOIN usuario

## Fase 3 — ChatWidget
- [ ] Crear `src/components/ChatWidget.tsx`
  - [ ] Botón flotante fixed bottom-right con badge usuarios en línea
  - [ ] Modal full-screen con overlay
  - [ ] Lista mensajes (scroll automático, flex-col-reverse)
  - [ ] Mensajes propios a la derecha
  - [ ] Columna usuarios en línea (desktop) / contador (mobile)
  - [ ] Input + botón enviar + Enter para enviar
  - [ ] Polling cada 3s con `?since=` cuando modal abierto
  - [ ] Cerrar con ESC o click fuera

## Fase 4 — Integración AppShell
- [ ] Modificar `src/components/AppShell.tsx` — agregar `<ChatWidget />` al final (solo si user aprobado)

## Fase 5 — Stats Mobile Compactas
- [ ] Modificar `src/app/(app)/dashboard/page.tsx`
  - [ ] Añadir barra `sm:hidden` con 4 celdas compactas
  - [ ] Agregar `hidden sm:grid` a los cards actuales

## Fase 6 — Build y deploy
- [ ] TypeScript check: `npx tsc --noEmit`
- [ ] Build: `NEXT_BUILD_STANDALONE=1 pnpm build`
- [ ] Deploy: `pm2 reload elitepass-mundial --update-env`
- [ ] Bump version `package.json` → v1.1.111
- [ ] Commit + push GitHub
- [ ] Sync openspec → GitHub → VM00
