# Plan de Cambios: Auditoría de Seguridad - ElitePass Mundial

**Fecha:** 2026-06-12  
**Basado en:** [[AUDIT_CODE_SECURITY_2026]]  
**Estado:** 🔴 CRITICAL FIXES PENDING

---

## TAREAS CRÍTICAS (Hoy - 1 día)

### TASK-001: Rotar Secretos Comprometidos
```
Severidad: CRÍTICA
Tiempo Estimado: 2 horas
Bloqueante: SÍ
Dependencias: Acceso a producción 10.0.0.4:5001
```

**Checklist:**
- [ ] Coordinar con DevOps acceso a prod
- [ ] Generar nuevos secretos para cada variable:
  - `DB_PASSWORD` → Nueva contraseña PostgreSQL
  - `JWT_SECRET` → Nueva random 64 chars
  - `SYNC_SECRET` → Nueva random 64 chars
  - `FOOTBALL_API_KEY` → Regenerar en proveedor
  - `MAIL_GRAPH_CLIENT_SECRET` → Regenerar en Azure
  - `VAPID_PRIVATE_KEY` → Regenerar con web-push
  - `CF_API_TOKEN` → Regenerar en Cloudflare
  - `AZURE_STORAGE_CONNECTION_STRING` → Regenerar en Azure

- [ ] Actualizar `.env.production` en prod
- [ ] Verificar all services arrancando con nuevos secretos
- [ ] Verificar logs sin errores de auth

**Código/Comandos:**
```bash
# Generar nuevos secretos
openssl rand -base64 32  # Para API keys
openssl rand -hex 32     # Para JWT_SECRET

# Rotar DB password
ALTER USER postgres WITH PASSWORD 'NEW_PASSWORD';

# Verificar
docker logs app_1 | grep -i "error\|auth" | tail -20
```

---

### TASK-002: Remover .env.local del Historial Git
```
Severidad: CRÍTICA
Tiempo Estimado: 1 hora
Bloqueante: SÍ
```

**Checklist:**
- [ ] Backup del repo local
- [ ] Ejecutar git filter-branch:
```bash
git filter-branch --force --tree-filter 'rm -f .env.local' -- --all
```
- [ ] Forzar push a origin (⚠️ destructive):
```bash
git push origin --force --all
```
- [ ] Verificar .env.local no aparece en historial:
```bash
git log --all --full-history -- .env.local
```
- [ ] Notificar a todo el equipo hacer pull fresh

---

## VULNERABILIDADES ALTAS (Próxima Sprint)

### TASK-003: Reducir JWT Session TTL a 2 Horas
```
Severidad: ALTA
Tiempo Estimado: 2 horas
Bloqueante: ⚠️ Afecta identity-sync
Dependencia: TASK-004 (coordinación identity)
```

**Archivo:** `/src/lib/auth.ts`

**Cambio:**
```typescript
// ANTES:
const SESSION_TTL_SECONDS = 60 * 60 * 12;  // 12 horas

// DESPUÉS:
const SESSION_TTL_SECONDS = 60 * 60 * 2;   // 2 horas
const REFRESH_TTL_SECONDS = 60 * 60 * 24 * 7;  // 7 días
```

**Checklist:**
- [ ] Coordinar con identity team ANTES de cambios
- [ ] Actualizar identity-sync.ts para soportar refresh tokens
- [ ] Testing: usuarios deben poder usar app 2h sin logout
- [ ] Testing: refresh token debe renovar sesión
- [ ] Verificar cambios en identity callback

---

### TASK-004: Fix Open Redirect en Auth Callback
```
Severidad: ALTA
Tiempo Estimado: 1 hora
Bloqueante: ⚠️ Identity callback
Dependencia: TASK-003
```

**Archivo:** `/src/app/api/auth/identity-callback/route.ts` líneas 154-156

**Cambio:**
```typescript
// ANTES:
if (redirectTo.startsWith('http')) {
  return NextResponse.redirect(redirectTo);
}

// DESPUÉS:
const ALLOWED_REDIRECTS = ['/dashboard', '/admin', '/perfil', '/'];
const isAllowedRedirect = ALLOWED_REDIRECTS.some(path => 
  redirectTo === path || redirectTo.startsWith(path + '?')
);

if (!isAllowedRedirect) {
  return NextResponse.redirect('/dashboard');
}
```

**Testing:**
- [ ] Redirect a `/dashboard` → ✅ funciona
- [ ] Redirect a `https://evil.com` → ❌ rechazado → `/dashboard`
- [ ] Redirect a `/admin?tab=users` → ✅ funciona
- [ ] Redirect a `//evil.com` → ❌ rechazado

---

### TASK-005: Fix XSS en Auth Callback
```
Severidad: ALTA
Tiempo Estimado: 1 hora
Bloqueante: SÍ - Identity callback
Dependencia: TASK-004
```

**Archivo:** `/src/app/api/auth/identity-callback/route.ts` líneas 160-197

**Cambio:**
```typescript
// ANTES:
html: `
  <script>
    window.sessionToken = '${sessionToken}';
    window.redirectTo = '${dest}';
  </script>
`

// DESPUÉS:
html: `
  <script>
    window.sessionToken = ${JSON.stringify(sessionToken)};
    window.redirectTo = ${JSON.stringify(dest)};
  </script>
`
```

**Testing:**
- [ ] Tokens con caracteres especiales se escapan
- [ ] No se ejecuta código malicioso
- [ ] XSS payload en redirectTo → neutralizado

---

### TASK-006: Fix /api/sync Authentication
```
Severidad: ALTA
Tiempo Estimado: 1.5 horas
Bloqueante: NO
```

**Archivo:** `/src/app/api/sync/route.ts`

**Cambio:**
```typescript
// ANTES:
const key = req.nextUrl.searchParams.get('key');
if (key !== secret) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

// DESPUÉS:
const authHeader = req.headers.get('Authorization');
const token = authHeader?.split(' ')[1];

// Timing-safe comparison
const isValid = crypto.timingSafeEqual(
  Buffer.from(token || ''),
  Buffer.from(secret)
);
if (!isValid) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
```

**Testing:**
- [ ] GET /api/sync sin header → 401
- [ ] GET /api/sync con header válido → 200
- [ ] Secreto no aparece en logs

---

## HARDENING SECURITY (Días 3-4)

### TASK-007: CSP Hardening (Remover unsafe-*)
```
Severidad: ALTA
Tiempo Estimado: 3 horas
Bloqueante: NO
Depende de: Verificar todos los inline scripts
```

**Archivo:** `/src/proxy.ts` línea 53

**Cambio:**
```javascript
// ANTES:
'Content-Security-Policy': "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'..."

// DESPUÉS:
'Content-Security-Policy': "default-src 'self'; script-src 'self' 'nonce-${nonce}'..."
```

**Proceso:**
1. [ ] Auditar todos `<script>` inline en proyecto
2. [ ] Reemplazar con nonces dinámicos
3. [ ] Verificar third-party scripts
4. [ ] Testing en Chrome, Firefox, Safari, Edge
5. [ ] Verificar console logs sin CSP warnings

---

### TASK-008: Rate Limiting Updates
```
Severidad: MEDIA
Tiempo Estimado: 1 hora
Bloqueante: NO
```

**Archivo:** `/src/proxy.ts`

**Cambio:**
```javascript
// ANTES:
'/api/auth': { windowMs: 60 * 1000, max: 10 }  // 10 req/min
'/api/auth/register': { windowMs: 60 * 1000, max: 5 }  // 5 req/min

// DESPUÉS:
'/api/auth': { windowMs: 60 * 1000, max: 3 }  // 3 req/min
'/api/auth/register': { windowMs: 60 * 1000, max: 2 }  // 2 req/min
```

**Testing:**
- [ ] Login normal funciona (1 request)
- [ ] 4tos requests rechazados
- [ ] Error 429 Too Many Requests

---

### TASK-009: File Validation en Payments
```
Severidad: MEDIA
Tiempo Estimado: 1.5 horas
Bloqueante: NO
```

**Archivo:** `/src/app/api/admin/payments/route.ts` líneas 137-182

**Cambio:**
```typescript
// Agregar validación:
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'application/pdf'];

const validateFile = (file: File) => {
  if (file.size > MAX_FILE_SIZE) throw new Error('Archivo muy grande');
  if (!ALLOWED_MIME_TYPES.includes(file.type)) throw new Error('Tipo archivo no permitido');
  
  // Validar contenido real (no solo extensión)
  const magic = await file.arrayBuffer().then(buf => new Uint8Array(buf).slice(0, 4));
  // Verificar PNG/JPG/PDF headers
};
```

---

## CODE QUALITY (Semanas 2-3)

### TASK-010: Crear Tipos Base
```
Tiempo Estimado: 3 horas
```

**Archivos a crear:**
- `/src/types/admin.ts` - Admin-related types
- `/src/types/auth.ts` - Auth types
- `/src/types/api.ts` - API response types

---

### TASK-011: Migrar `any` Types
```
Tiempo Estimado: 15-20 horas
Prioridad: Después de security fixes
```

### TASK-012: Dividir admin/page.tsx
```
Tiempo Estimado: 8-10 horas
Prioridad: Después de types
```

---

## DEFINICIÓN DE HECHO GLOBAL

```gherkin
Given: Auditoría de seguridad completada
When: Todos los tasks implementados y aprobados
Then:
  ✅ Cero secretos en repositorio
  ✅ JWT TTL ≤ 2 horas
  ✅ CSP sin 'unsafe-inline' o 'unsafe-eval'
  ✅ No open redirects posibles
  ✅ Auth callbacks con XSS protection
  ✅ /api/sync usa Bearer tokens
  ✅ Rate limiting < 3 req/min
  ✅ File validation en uploads
  ✅ any types < 10 en codebase
  ✅ Segunda auditoría pasa
  ✅ Todos los tests pasan
  ✅ Performance metrics mantenidas
```

---

**Plan Creado:** 2026-06-12  
**Estado:** 🔴 PENDIENTE APROBACIÓN SENIOR DEV
**Estimado Total:** 50 horas
**Equipo Recomendado:** 2-3 developers full-time por 1 semana
