# Tasks: Refactor page.tsx → App Router pages + Push Notifications

## Refactor Next.js App Router pages
- [x] Crear `src/app/(app)/layout.tsx` — AuthGuard + AppShell wrapper
- [x] Crear `src/contexts/AppContext.tsx` — user, notifications, SSE, push, toast
- [x] Crear `src/components/AppShell.tsx` — nav desktop/mobile, goal overlay, toast
- [x] Crear `src/app/(app)/dashboard/page.tsx` — hero stats, partidos en vivo, próximos, noticias
- [x] Crear `src/app/(app)/partidos/page.tsx` — lista de partidos con pronósticos
- [x] Crear `src/app/(app)/fixture/page.tsx` — bracket / standings / eliminatoria
- [x] Crear `src/app/(app)/ranking/page.tsx` — leaderboard (participantes + visores)
- [x] Crear `src/app/(app)/reglas/page.tsx` — reglas / about
- [x] Crear `src/app/(app)/perfil/page.tsx` — perfil, passkeys, push, stats
- [x] Crear `src/app/(app)/admin/page.tsx` — usuarios, empresa, mensajes (admin/superadmin)
- [x] Refactorizar `src/app/page.tsx` → solo redirección (autenticado→/dashboard, sin auth→Identity SSO)
- [x] Extraer constantes a `src/lib/constants.ts` (TEAM_CODES, getTeamFlag, PHASES_APUESTA, DEFAULT_MODOS_POR_FASE)
- [x] Verificar TypeScript (`npx tsc --noEmit`) — sin errores

## Push Notifications automáticas
- [x] Crear tabla `match_notif_log` (match_id, event, sent_at) para deduplicación
- [x] Agregar `sendPushToAllActive()` en `src/lib/push.ts`
- [x] Agregar `sendPushToUsersWithoutPrediction()` en `src/lib/push.ts`
- [x] Modificar `src/lib/sync.ts`:
  - [x] `sendUpcomingReminders()` — avisos 60min y 30min antes del partido (solo sin pronóstico)
  - [x] Notificación automática al iniciar partido (`event: live`)
  - [x] Notificación automática al meter gol (`event: goal_X_X`)
  - [x] Notificación automática al terminar partido + tabla de líderes (`event: finished`)
  - [x] Insertar registros en `notifications` table para cada evento (aparece en pestaña mensajes)
  - [x] `broadcastUpdate('notification', {auto:true})` para refrescar pestaña en tiempo real

## Animación de gol (full screen)
- [x] Modificar `AppShell.tsx` — overlay full-screen con anillos ping, blur, tarjeta animada
- [x] Modificar `AppContext.tsx` bootstrap — detectar goles recientes al entrar (últimos 5 min), mostrar con `missed: true`
