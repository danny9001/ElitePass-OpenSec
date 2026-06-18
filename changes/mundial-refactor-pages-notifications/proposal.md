# Proposal: Refactor Monolith + Push Notifications Mundial 2026

## Problema
`src/app/page.tsx` era un archivo monolítico de ~5900 líneas con todos los tabs del app en un solo componente client. Esto causaba:
- Tiempo de carga lento (todo el JS en una página)
- Difícil mantenimiento
- Sin separación de rutas para navegación directa

Además, las notificaciones push no se enviaban automáticamente en eventos del partido (inicio, gol, finalización, recordatorios).

## Solución
1. Extraer cada tab en su propia página bajo `src/app/(app)/` con Next.js App Router
2. Shared state via `AppContext` (React Context)
3. Layout compartido en `AppShell.tsx` con navegación desktop/mobile
4. Auto-push en `sync.ts` usando `match_notif_log` para deduplicación
5. Animación full-screen al detectar gol (vía SSE o al cargar app con gol reciente)
