# Tasks: Fix Stale Score Notifications

## Estado: COMPLETADO ✅

- [x] Agregar tipo `PendingGoalNotif` al top de `sync.ts`
- [x] Agregar import `logSystem` de `./mail`
- [x] Modificar firma de `sync365Scores` (parámetro opcional map)
- [x] Reemplazar bloque goal+finished de 365Scores con acumulador
- [x] Modificar firma de `syncFixtureDownload` (parámetro opcional map)
- [x] Reemplazar bloque goal+finished de FixtureDownload con acumulador
- [x] Modificar firma de `syncESPNScoreboard` (parámetro opcional map)
- [x] Reemplazar bloque goal+finished de ESPN con acumulador
- [x] Modificar firma de `syncFootballData` (parámetro opcional map)
- [x] Reemplazar bloque goal+finished de FootballData con acumulador
- [x] Agregar función `flushPendingNotifications()` antes de `syncMatches()`
- [x] Modificar `syncMatches()`: crear map, pasar a fuentes, llamar flush, crear tabla score_change_log
- [x] TypeScript check sin errores (`npx tsc --noEmit`)
- [x] Crear documentos OpenSpec (proposal, design, tasks)

## Verificación

```sql
-- Ver detecciones por fuente durante un partido en vivo
SELECT * FROM score_change_log WHERE match_id = X ORDER BY created_at;

-- Ver logs de sync (detecciones + push)
SELECT * FROM system_logs WHERE categoria = 'SYNC' ORDER BY created_at DESC LIMIT 50;

-- Verificar que solo hay UNA notif por score (no duplicados)
SELECT * FROM match_notif_log WHERE match_id = X ORDER BY sent_at DESC;
```
