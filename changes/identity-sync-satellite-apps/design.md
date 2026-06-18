# Design: Sincronización automática apps satélite → Identity

## Decisiones Técnicas

### 1. Endpoint POST /api/v1/sync en identity
- **Auth:** `X-App-Secret` header (mismo secreto que usan las apps para todas las llamadas a identity)
- **Acciones:** discriminadas por campo `action` en el body
- **Idempotente:** usa `findUnique` + `create` — ejecutar N veces es seguro
- **Archivo:** `src/app/api/v1/sync/route.ts`

```
POST /api/v1/sync
{
  "action": "sync-user-empresa" | "sync-empresa",
  "appSlug": "pos" | "reservas",
  "email": "...",           // solo en sync-user-empresa
  "empresaSlug": "...",
  "empresaNombre": "...",
  "rol": "member"
}
```

### 2. Cliente en apps satélite — fire-and-forget
- La llamada a identity es `void` — no bloquea la respuesta de la app
- Los errores (identity caído, timeout) solo generan `console.warn`, nunca rompen la operación
- **Razón:** identity es un sistema auxiliar; crear un usuario en POS/Reservas no puede fallar por un timeout de red interno

### 3. Flujo de sincronización
```
App (POS/Reservas)
  → crea usuario/empresa en su DB propia
  → responde 201 al cliente (no espera identity)
  → [async] identityFetch('/api/v1/sync', { ... })
       → identity: ensureEmpresa + ensureLicencia + ensureUsuario
```

### 4. Optimizaciones incluidas en este deploy
- **Reservas:** `optimizePackageImports` para lucide-react, recharts, date-fns, radix-ui → tree-shaking agresivo
- **Reservas:** `compress: true` + `poweredByHeader: false` en next.config.js
- **Payments:** `compression` middleware Express level 6 (~60-70% reducción de payload)
- **POS landing:** 4 apps en grid (Reservas, POS, Mundial, Identity) + dist reconstruido

### 5. Nota sobre ecosystem.config.js del POS
El archivo ya estaba trackeado en git. Contiene secretos actualizados (JWT, Telegram API key, contraseña admin). **Pendiente:** mover secretos a `.env` y sacar del tracking git (aplica regla de seguridad OpenSpec).
