# Fix: SEC-01 + SEC-02 — Error Leakage + CSP Nonce
## ElitePass Mundial — v1.1.88 — 2026-06-19

**Commit:** `9e1deb8`  
**Tipo:** Security hardening — 2 fixes críticos  

---

## SEC-01: Eliminar `error.message` de responses de producción

### Problema
20 rutas API devolvían `error.message` (o variantes como `'Error del servidor: ' + error.message`) directamente en la respuesta JSON. Esto expone:
- Nombres de tablas de PostgreSQL
- Columnas de DB
- Queries SQL fallidas
- Rutas de archivo del servidor
- Detalles internos de Node.js

### Solución
Todos los `catch` blocks de API routes ahora devuelven un mensaje genérico al cliente y logean el error completo solo en `console.error` (server-side):

```typescript
// ANTES ❌
catch (error: any) {
  return NextResponse.json({ error: error.message }, { status: 500 });
}

// DESPUÉS ✅
catch (error: any) {
  console.error('[route-name] Error:', error); // Log completo en servidor
  return NextResponse.json({ error: 'Error del servidor' }, { status: 500 });
}
```

### Archivos modificados (20 routes)
- `src/app/api/users/route.ts`
- `src/app/api/auth/webauthn/passkeys/route.ts` (×2)
- `src/app/api/notifications/route.ts` (×3)
- `src/app/api/profile/route.ts` (×2)
- `src/app/api/profile/payments/route.ts`
- `src/app/api/sync/route.ts` (×2)
- `src/app/api/news/route.ts`
- `src/app/api/companies/route.ts`
- `src/app/api/settings/route.ts` (×2)
- `src/app/api/groups/route.ts`
- `src/app/api/matches/route.ts`
- `src/app/api/predictions/route.ts`
- `src/app/api/admin/logs/route.ts`
- `src/app/api/admin/recalculate/route.ts`
- `src/app/api/admin/sync/route.ts` (×2)
- `src/app/api/admin/payments/route.ts`
- `src/app/api/admin/stats/route.ts`
- `src/app/api/stats/me/route.ts`
- `src/app/api/health/route.ts` → responde `{ status: 'unhealthy', error: 'Database unavailable' }`

---

## SEC-02: CSP Nonce en lugar de `unsafe-inline`

### Problema
`proxy.ts` usaba `'unsafe-inline'` en la directiva `script-src` del Content Security Policy (CSP) en producción. Esto **invalida completamente la protección XSS** del CSP — un atacante que inyecte un `<script>` inline puede ejecutar código sin restricción.

La causa raíz: el registro del Service Worker en `layout.tsx` usaba `dangerouslySetInnerHTML` (inline script) sin nonce.

### Solución

**1. `src/proxy.ts`** — Generar nonce por request, eliminiar unsafe-inline:
```typescript
// Nonce único por request — base64 de un UUID
const nonce = Buffer.from(crypto.randomUUID()).toString('base64');

// Forward nonce al Server Component via request header
const res = NextResponse.next({
  request: {
    headers: new Headers({ ...Object.fromEntries(req.headers), 'x-nonce': nonce }),
  },
});

// CSP: nonce en lugar de unsafe-inline
const scriptSrc = isProd
  ? `script-src 'self' 'nonce-${nonce}' https://static.cloudflareinsights.com`
  : `script-src 'self' 'nonce-${nonce}' 'unsafe-eval' https://static.cloudflareinsights.com`;
```

**2. `src/app/layout.tsx`** — Leer nonce y aplicar al script del SW:
```typescript
// layout.tsx ahora es async (Server Component)
export default async function RootLayout({ children }) {
  const nonce = (await headers()).get('x-nonce') ?? '';

  return (
    <html>
      <head>
        <script
          nonce={nonce}  // ← El browser valida este nonce contra el CSP
          dangerouslySetInnerHTML={{ __html: `...service worker registration...` }}
        />
      </head>
    </html>
  );
}
```

### Cómo funciona el nonce
1. Middleware (proxy.ts) genera nonce aleatorio en cada request
2. Lo mete en el CSP header: `script-src 'nonce-abc123'`
3. Lo pasa como `x-nonce` header a Next.js Server Components
4. Next.js aplica automáticamente el nonce a sus propios scripts de hidratación
5. layout.tsx lo aplica al script inline del SW
6. Browser ejecuta SOLO scripts con el nonce correcto — todos los demás bloqueados

### Nota: `unsafe-inline` en `style-src` sigue siendo aceptable
`style-src 'unsafe-inline'` permanece para CSS (Tailwind inline styles). CSS no puede ejecutar JS y el riesgo de CSS injection es significativamente menor. Este es el estándar de la industria para apps con Tailwind.

---

## Verificación

```bash
# SEC-01: No debe haber error.message en responses de producción
grep -rn "error\.message" src/app/api/ | grep -v "console\." | grep "return NextResponse"
# → vacío ✅

# SEC-02: No debe haber unsafe-inline en script-src (solo en style-src)
grep "unsafe-inline" .next/server/chunks/[root-of-the-server]*.js
# → solo 1 match = style-src ✅

# Build exitoso con proxy registrado
pnpm build | tail -5
# → ƒ Proxy (Middleware) ✅
```

---

## Estado de Seguridad Post-Fix

| Issue | Antes | Después |
|-------|-------|---------|
| SEC-01 error.message | ❌ 26 leaks en 17 rutas | ✅ 0 leaks |
| SEC-02 CSP unsafe-inline | ❌ script-src unsafe-inline | ✅ script-src nonce-{uuid} |
| Middleware registro | ⚠️ vacío en manifest | ✅ ƒ Proxy (Middleware) activo |
