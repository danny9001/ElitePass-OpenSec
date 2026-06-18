# Design: Refactor Monolito → Páginas Separadas

## Estrategia: "Shell + Páginas"

```
/                    → redirect a /dashboard (si autenticado)
/dashboard           → Tab Inicio
/partidos            → Tab Partidos  
/fixture             → Tab Fixture / Posiciones
/ranking             → Tab Ranking
/reglas              → Tab Reglas
/perfil              → Tab Perfil
/admin               → Tab Admin (ya tiene sub-tabs propios)
/admin/predictions   → Ya existe ✓
/notifications       → Ya existe ✓
/tv                  → Ya existe ✓
/invitacion          → Ya existe ✓
```

## Shared Infrastructure a extraer (nuevos archivos)

### `src/lib/constants.ts`
Mover desde page.tsx:
- `TEAM_CODES`, `getTeamFlag()`, `formatPlaceholderText()`
- `PHASES_APUESTA`, `DEFAULT_MODOS_POR_FASE`

### `src/components/AppShell.tsx`
La nav top + bottom nav que hoy está en page.tsx. Recibe `user` y `activeRoute` como props. Contiene los botones de navegación (cambian de `setActiveTab` a `<Link href="/ruta">`).

### `src/hooks/useSession.ts`
```ts
// Reemplaza el bloque checkSession en page.tsx
export function useSession() {
  // fetch /api/auth → { user } 
  // redirect a / si no autenticado
  return { user, loading }
}
```

### `src/hooks/useRealtime.ts`
```ts
// Encapsula el EventSource de /api/realtime
export function useRealtime(onEvent: (type, data) => void) { ... }
```

### `src/hooks/useMatches.ts`
```ts
// fetch /api/matches + /api/predictions
export function useMatches() { return { matches, predictions, loading } }
```

## Flujo de Auth
Cada página llama `useSession()`. Si no hay sesión → redirect `/` (login). El login en `/` redirige a `/dashboard` tras éxito.

## Estado compartido — ¿Context o fetch por página?
**Recomendación: fetch por página** (KISS). Cada ruta fetcha lo que necesita:
- `/dashboard` → matches (3 próximos), leaderboard (top 5), predictions propias
- `/partidos` → matches completo + predictions propias  
- `/fixture` → matches completo
- `/ranking` → leaderboard completo + companies
- `/perfil` → user, stats, passkeys
- `/admin` → adminUsers, companies, groups, notifications

Esto elimina la necesidad de Context global y cada página es independiente.

## Complejidad del modal de apuesta
El `BetModal` (modal para ingresar pronóstico) es compartido hoy. Se extrae a `src/components/BetModal.tsx` y se importa en `/partidos` y `/dashboard`.

## Estimación de reducción de bundle
- page.tsx actual: ~5 900 líneas = ~240KB JS minificado estimado  
- Dashboard (la página más visitada): ~800 líneas → ~30KB
- Code splitting automático de Next.js hace el resto

## Compresión (ya habilitada)
`compress: true` en next.config.ts activa gzip en el servidor Next.js. Dado que hay nginx en frente, también se puede habilitar gzip en nginx para servir assets estáticos comprimidos.
