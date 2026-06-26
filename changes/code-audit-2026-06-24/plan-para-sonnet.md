# Auditoría de Código + Plan de Remediación — ElitePass Mundial
**Auditor:** Opus 4.8 · **Ejecutor:** Sonnet 4.6 · **Fecha:** 2026-06-24

> Plan diseñado para que Sonnet lo ejecute paso a paso. Cada ítem tiene:
> archivo exacto, qué hacer, y cómo verificar. **NO romper funcionalidad existente.**
> Versionar `package.json` a 1.1.XXX en cada commit (regla del proyecto).

---

## 🔴 CRÍTICO — Hacer HOY, en este orden

### C1 — Secreto SSO filtrado en git (forja de superadmin) ⚠️ EXPLOIT CONFIRMADO
**Archivo:** `test_token.js` (raíz, trackeado en git desde commit `d24c31b`)
**Problema:** Contiene el `IDENTITY_JWT_SECRET` de producción en texto plano y demuestra
cómo firmar un token con `role: 'superadmin'`. La ruta `identity-callback/route.ts:49`
hace `jwt.verify(token, IDENTITY_JWT_SECRET)` y confía en `payload.role === 'superadmin'`.
**Cualquiera con acceso de lectura al repo puede entrar como superadmin.**

**Acción (en orden estricto):**
1. **Rotar `IDENTITY_JWT_SECRET`** en `.env.local` Y en el servicio Identity SSO (coordinar — ambos lados deben cambiar a la vez o se rompe el login SSO). Generar: `openssl rand -base64 32`.
2. `git rm test_token.js` y commit.
3. Purgar de la historia: `git filter-repo --path test_token.js --invert-paths` (o BFG). Esto reescribe historia → coordinar push forzado.
4. Agregar `test_token.js` y `*token*.js` de prueba al `.gitignore`.
5. Reiniciar PM2 con el nuevo env.

**Verificar:** `git log --all --oneline -- test_token.js` no devuelve nada; login SSO sigue funcionando; el token viejo ya no valida.

---

### C2 — `SYNC_SECRET`/`SCHEDULER_SECRET` con fallback débil y en query string
**Archivos:** `src/app/api/sync/route.ts:26`, `scheduler.js:12,45`
**Problema:** `const secret = process.env.SYNC_SECRET || 'sync2026'`. Si el env falta,
el secreto es público y predecible. Además el scheduler lo manda como `?key=...` →
queda registrado en logs de acceso de nginx.

**Acción:**
1. Eliminar el fallback: si `process.env.SYNC_SECRET` no existe → `throw` / responder 500. Nunca usar valor por defecto.
2. Confirmar `SYNC_SECRET` y `SCHEDULER_SECRET` definidos en `.env.local` (rotar si fueron `sync2026`).
3. Cambiar el scheduler para enviar el secreto por header `Authorization: Bearer` en lugar de query string. La ruta `sync/route.ts` ya soporta Bearer (verificar) — usar solo ese camino y quitar el soporte de `?key=`.

**Verificar:** `curl /api/sync` sin auth → 401; con Bearer correcto → 200; logs de nginx ya no contienen el secreto.

---

### C3 — Telegram: API key hardcoded + IP interna por HTTP plano
**Archivo:** `src/lib/push.ts:26-27`
**Problema:** `const apiKey = 'a42fa223449069d6825989f8206335ca'` y
`fetch('http://10.0.0.4:3200/...')` (sin TLS, credencial en código).

**Acción:**
1. Mover la key a `process.env.TELEGRAM_GATEWAY_KEY` y la URL a `process.env.TELEGRAM_GATEWAY_URL`.
2. Agregarlas a `.env.local`. Rotar la key en el gateway Telegram.
3. Si el gateway está en la misma red privada, documentar por qué HTTP es aceptable; si sale de la red, usar HTTPS.

**Verificar:** `grep -rn "a42fa223\|10.0.0.4" src/` vacío; notificaciones Telegram siguen llegando.

---

## 🟠 ALTO

### A1 — Sin rate limiting (brute force en login/register)
**Archivos:** `src/app/api/auth/route.ts`, `auth/register/route.ts`, `predictions/route.ts`
**Acción:** Implementar rate limiting simple en DB o memoria por IP+email.
Crear helper `src/lib/rate-limit.ts` (tabla `rate_limits` o `Map` en memoria con TTL).
Límites sugeridos: login 5/min por IP, register 3/hora por IP. Devolver 429 al exceder.
**Verificar:** 6 logins fallidos seguidos → el 6º devuelve 429.

### A2 — Validación numérica débil (predictions / matches)
**Archivos:** `src/app/api/predictions/route.ts` (líneas 84, 110-126, 248-249),
`src/app/api/matches/route.ts:107-108`
**Problema:** `parseInt(predLocal)` sin validar → `NaN` (→ NULL en DB) o valores absurdos (999999).
**Acción:** Crear validador en `src/lib/validation.ts`:
```ts
export function validateScore(v: unknown): number | null {
  const n = Number(v);
  return Number.isInteger(n) && n >= 0 && n <= 99 ? n : null;
}
```
Usarlo en predictions (batch + individual) y matches; si devuelve `null` → rechazar con 400.
**Verificar:** enviar `predLocal: "abc"` o `-5` o `999` → 400.

### A3 — Schema drift: 7 tablas runtime ausentes de `init.sql`
**Problema:** `system_logs`, `score_change_log`, `pending_downgrades`, `user_presence`,
`match_notif_log`, `mail_queue`, `audit_logs` se usan pero NO están en `init.sql`.
Un deploy fresco desde `init.sql` arranca roto.
**Acción:** Agregar las 7 definiciones `CREATE TABLE IF NOT EXISTS ...` a `init.sql`
(copiar los esquemas reales desde la DB: `pg_dump --schema-only -t <tabla>`).
`init.sql` debe ser la fuente de verdad del esquema.
**Verificar:** `init.sql` en una DB vacía de prueba crea todas las tablas sin error.

### A4 — DDL en cada request
**Archivos:** ~10 rutas con `CREATE TABLE IF NOT EXISTS` por request (notifications, companies, groups, settings, push/subscribe, predictions, admin/users, admin/payments, etc.)
**Acción:** Tras completar A3, eliminar los `CREATE TABLE` de los handlers. El esquema
vive en `init.sql` y migraciones, no en el hot path.
**Verificar:** la app funciona sin los `CREATE TABLE` inline (DB ya tiene las tablas).

---

## 🟡 MEDIO

### M1 — 361 errores ESLint (mayoría `no-explicit-any`)
**Acción incremental (no bloquear):** Empezar por `src/lib/` (núcleo): tipar
`match-utils.ts`, `push.ts`, `realtime.ts`, `mail.ts`, los `catch (err: any)` de `sync.ts`.
Definir interfaces reales (ej. `Match`, `Prediction`, `UserRow`). No hacer todo de golpe;
hacer PRs por módulo. `pnpm lint` debe ir bajando el conteo.

### M2 — SSE no cruza procesos PM2 (eventos al 25% de clientes)
**Archivo:** `src/lib/realtime.ts` (EventEmitter in-process) + 4 instancias en `ecosystem.config.js`
**Acción:** Reemplazar el EventEmitter por Postgres `LISTEN/NOTIFY` (ya hay pool pg) o Redis pub/sub.
Cada instancia escucha el canal y reenvía a sus clientes SSE. Mantener el polling de 90s como fallback.
**Verificar:** disparar sync en una instancia → clientes de las 4 instancias reciben el evento.

### M3 — `console.log` filtra datos en auth.ts
**Archivo:** `src/lib/auth.ts:31,40,45,55,59`
**Acción:** Quitar o degradar a `console.debug` detrás de `if (process.env.DEBUG_AUTH)`.
No registrar IDs de usuario ni motivos de fallo JWT en producción.

### M4 — sync sin mutex (solapamiento concurrente)
**Archivo:** `src/lib/sync.ts` (`syncMatches`)
**Acción:** Envolver `syncMatches` con `pg_try_advisory_lock(<clave fija>)`. Si no se obtiene
el lock, salir temprano (otro sync corriendo). Liberar al final. Patrón ya usado en `mail.ts`.

### M5 — sync.ts 1813 líneas con 4 bloques duplicados
**Acción (refactor, baja prioridad):** Extraer el bloque común goal/finished/downgrade
a un helper `processMatchUpdate(localMatch, parsed, accumulators)` que las 4 fuentes invoquen.
Reduce ~600 líneas duplicadas. Hacer con tests antes/después para no regresar.

---

## 🔵 BAJO / INCOHERENCIAS

- **B1:** `CLAUDE.md` dice "JWT TTL 12h"; real es 7 días (`auth.ts:19`, 604800s). Actualizar doc (y la memoria que lo referencia).
- **B2:** `notifSent` definido y nunca usado (`sync.ts:25`). Eliminar.
- **B3:** categoría log `'payment'` en minúscula vs resto en MAYÚSCULA → no aparece en StatsTab. Normalizar a `'PAGO'` (`admin/payments/route.ts:196`).
- **B4:** Cero tests (DES-01). Añadir Vitest + tests para `validation.ts`, `match-utils.ts` (puntajes), y el flujo de score-correction/downgrade.

---

## Orden de ejecución recomendado para Sonnet
1. **C1 → C2 → C3** (seguridad crítica, mismo día). C1 requiere coordinar rotación con el equipo Identity.
2. **A2 + A1** (validación + rate limit — protegen escritura/abuso).
3. **A3 → A4** (schema source-of-truth, luego quitar DDL del hot path).
4. **M3, M4, B1, B2, B3** (rápidos, bajo riesgo).
5. **M1** (tipado incremental por módulo).
6. **M2** (LISTEN/NOTIFY — cambio de arquitectura, probar bien).
7. **B4 → M5** (tests primero, luego refactor de sync.ts respaldado por esos tests).

## Reglas
- Un commit por ítem (o grupo pequeño), versión `1.1.XXX` incremental en `package.json`.
- `npx tsc --noEmit` y `pnpm build` deben pasar antes de cada deploy.
- No tocar la lógica de scoring ni el sync funcionando salvo lo indicado.
- Deploy: `pnpm build && pm2 reload elitepass-mundial --update-env` + purga CF.
