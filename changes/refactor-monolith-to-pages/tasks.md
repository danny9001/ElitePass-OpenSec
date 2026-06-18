# Tasks: Refactor Monolito → Páginas Separadas

## Fase 1 — Infraestructura compartida
- [ ] Crear `src/lib/constants.ts` con TEAM_CODES, getTeamFlag, formatPlaceholderText, PHASES_APUESTA, DEFAULT_MODOS_POR_FASE
- [ ] Crear `src/hooks/useSession.ts` — encapsula checkSession + redirect
- [ ] Crear `src/hooks/useRealtime.ts` — encapsula EventSource SSE
- [ ] Crear `src/hooks/useMatches.ts` — fetch matches + predictions
- [ ] Crear `src/components/BetModal.tsx` — modal de pronóstico extraído
- [ ] Crear `src/components/AppShell.tsx` — top nav + bottom nav con Link en vez de setActiveTab

## Fase 2 — Páginas nuevas (una por una, sin romper la app)
- [ ] Crear `src/app/dashboard/page.tsx` — Tab Inicio
- [ ] Crear `src/app/partidos/page.tsx` — Tab Partidos + BetModal
- [ ] Crear `src/app/fixture/page.tsx` — Tab Fixture/Posiciones
- [ ] Crear `src/app/ranking/page.tsx` — Tab Ranking
- [ ] Crear `src/app/reglas/page.tsx` — Tab Reglas
- [ ] Crear `src/app/perfil/page.tsx` — Tab Perfil (passkeys, push, Telegram)
- [ ] Crear `src/app/admin/page.tsx` — Tab Admin (usuarios, empresa, mensajes)

## Fase 3 — Login / root
- [ ] Refactorizar `src/app/page.tsx` → solo pantalla de login + redirect a /dashboard
- [ ] Redirect automático `/` → `/dashboard` si hay sesión activa

## Fase 4 — Limpieza
- [ ] Eliminar todos los estados huérfanos de page.tsx
- [ ] Verificar que `npx tsc --noEmit` pasa sin errores
- [ ] Verificar que `pnpm build` completa exitosamente

## Notas de ejecución
- Hacer una página a la vez. No romper el tab existente en page.tsx hasta que la nueva página esté lista.
- Estrategia "parallel run": la nueva ruta /dashboard existe y funciona → recién entonces sacar el tab 'dashboard' de page.tsx.
- El BetModal es el componente más complejo a extraer — hacerlo en Fase 1 antes de las páginas.
