# Tasks: Sincronización automática apps satélite → Identity

## Identity
- [x] Crear `src/app/api/v1/sync/route.ts` con acciones sync-user-empresa y sync-empresa
- [x] Auth por X-App-Secret header
- [x] Lógica idempotente: ensureEmpresa + ensureLicencia + ensureUsuario
- [x] Build Next.js
- [x] Commit + push → 1.0.3
- [x] pm2 restart elitepass-identity

## POS
- [x] Crear `backend/src/config/identity-sync.ts` (cliente HTTP fire-and-forget)
- [x] `empresas.router.ts`: llamar syncEmpresa al crear empresa
- [x] `usuarios.router.ts`: llamar syncUserEmpresa al crear usuario
- [x] Commit + push → 1.1.8
- [x] pm2 restart elitepass-pos

## Reservas
- [x] `src/lib/identity-client.ts`: syncUserEmpresa()
- [x] `user-actions.ts`: llamar syncUserEmpresa en createUser (solo roles staff)
- [x] next.config.js: compress + poweredByHeader + optimizePackageImports
- [x] @next/bundle-analyzer dev dep
- [x] Build Next.js
- [x] Commit + push → 1.4.84
- [x] pm2 restart elitepass-reservas

## Payments
- [x] compression@1.8.1 instalado
- [x] app.ts: `app.use(compression({ level: 6 }))` antes de helmet
- [x] Commit + push → 1.0.3
- [x] pm2 restart elitepass-payments

## POS Landing
- [x] Agregar ElitePass Mundial (mundial.genial-it.net) al grid de apps
- [x] Grid cambiado a 2-col mobile / 4-col desktop
- [x] pnpm.yaml: onlyBuiltDependencies para esbuild (fix build pnpm v11)
- [x] Dist reconstruido (fix pantalla negra)
- [x] Commit + push

## Pendiente (no completado en este change)
- [ ] Mover secretos de ecosystem.config.js (POS) a archivo .env y sacar del tracking git
- [ ] Agregar apuestas-mundial-2026 al spec de antigravity
