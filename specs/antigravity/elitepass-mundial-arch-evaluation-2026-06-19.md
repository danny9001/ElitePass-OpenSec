# Evaluación de Arquitectura, Diseño y Seguridad
## ElitePass Mundial — 2026-06-19

**Versión app auditada:** 1.1.86  
**Stack:** Next.js 16 · React 19 · TypeScript · PostgreSQL (raw pool, sin ORM) · Tailwind v4  
**Deployment:** Podman cluster nginx-lb → app_1/app_2 → postgres  
**Evaluador:** Claude Sonnet 4.6 via skill-arch-security-evaluator  

---

## PILAR 1 — Arquitectura y Cohesión

### Diagnóstico Actual

**✅ LO QUE ESTÁ BIEN:**
- Refactorización del monolito (5.900 líneas en `page.tsx`) a páginas separadas con App Router — excelente evolución
- Estructura por feature/intención: `(app)/dashboard`, `(app)/partidos`, `(app)/fixture`, `(app)/ranking`, `(app)/perfil`, `(app)/admin`
- Capas bien separadas: `src/lib/` (lógica transversal) · `src/app/api/` (HTTP handlers) · `src/app/(app)/` (páginas) · `src/components/` (UI)
- `AppContext` centraliza estado compartido entre páginas sin prop-drilling
- Librerías bien nombradas: `auth.ts`, `db.ts`, `validation.ts`, `realtime.ts`, `mail.ts`
- `proxy.ts` como middleware centralizado para rate limiting + security headers → correcto

**❌ LO QUE ESTÁ FALLANDO:**

**1. `logSystem` vive en `mail.ts` — acoplamiento conceptual incorrecto**
El logging de sistema es un cross-cutting concern. Tenerlo en el módulo de mail acopla toda la app al módulo de email. Cualquier route que quiera loggear importa `mail.ts`.

**2. `ensureNotificationsTables()` duplicada en 3+ routes**  
La misma función con el mismo DDL aparece en:
- `src/app/api/admin/users/route.ts`
- `src/app/api/notifications/route.ts`  
- `src/app/api/admin/notify-scheduled/route.ts`
Las mismas tablas `notifications` y `notification_reads` se crean en 3 lugares distintos.

**3. DDL de runtime en API routes — 17 `CREATE TABLE IF NOT EXISTS` en 10+ archivos**
Se crean tablas en tiempo de request. Si la DB tiene un problema de permisos, la primera request falla con un error críptico.

**4. `admin/users/route.ts` es un God Controller (539 líneas, 7 actions en un solo POST)**  
Un solo endpoint maneja: `editUser`, `approve`, `deny`, `set_pending`, `assignCompany`, `toggleParticipa`, `setCompanies`, `create`. Este patrón violaría REST, pero en Next.js App Router es aceptable si se mantiene manejable. 539 líneas indica que ya superó ese umbral.

### Nivel de Riesgo: MEDIO

### Propuesta de Refactorización

```typescript
// ✅ Crear src/lib/logger.ts — desacoplar logSystem de mail
export async function logSystem(
  nivel: string, categoria: string, mensaje: string, detalles?: string
): Promise<boolean> {
  // misma implementación actual, movida desde mail.ts
}

// ✅ Crear src/lib/migrations.ts — DDL centralizado
export async function ensureSchema(): Promise<void> {
  // Todas las CREATE TABLE IF NOT EXISTS en un solo lugar
  // Llamar desde un init script o desde el health endpoint al arrancar
}
```

---

## PILAR 2 — Reglas de Diseño Simple (Kent Beck)

### 1. Pasa todos los tests — RIESGO ALTO

**Diagnóstico:** CERO archivos de test. El proyecto es "código de fe" — ninguna lógica de negocio tiene cobertura automatizada. La función `sanitizeText`, `isValidEmail`, `validatePassword`, la validación de apuestas por tiempo, el cálculo de cierre de apuestas — todo va a producción sin net de seguridad.

**Propuesta:**
```bash
pnpm add -D vitest @vitejs/plugin-react
```

```typescript
// src/lib/validation.test.ts
import { describe, it, expect } from 'vitest';
import { sanitizeText, isValidEmail, validatePassword } from './validation';

describe('sanitizeText', () => {
  it('strips HTML tags', () => {
    expect(sanitizeText('<script>alert(1)</script>nombre')).toBe('nombre');
  });
  it('strips XSS chars', () => {
    expect(sanitizeText('"><img onerror=alert(1) src=x>')).toBe('');
  });
  it('truncates at maxLen', () => {
    expect(sanitizeText('a'.repeat(200), 100)).toHaveLength(100);
  });
});

describe('isValidEmail', () => {
  it('rejects emails > 254 chars', () => {
    expect(isValidEmail('a'.repeat(250) + '@x.com')).toBe(false);
  });
  it('accepts valid email', () => {
    expect(isValidEmail('user@empresa.com')).toBe(true);
  });
});
```

### 2. Revela la intención — RIESGO BAJO-MEDIO

**✅ Bien:** `sanitizeText`, `isValidEmail`, `validatePassword`, `getSessionUser`, `broadcastUpdate`, `deviceLabelFromUA` — nombres altamente explícitos.

**❌ Mal:**
- `tipo` para roles de usuario → opaco, debería ser `role` (o al menos `rol`)
- `tincaso` — campo completamente opaco en `users` y en la API `/api/tincaso`
- `any[]` proliferado: `notifications: any[]`, `companies?: any[]`, `matches: any[]` en `AppContext`

**Propuesta:**
```typescript
// Reemplazar any[] por types explícitos en AppContext
interface Notification {
  id: number;
  titulo: string;
  contenido: string;
  tipo: 'info' | 'success' | 'error' | 'warn';
  leido: boolean;
  created_at: string;
}

interface Company {
  id: number;
  nombre: string;
  color: string;
  monto_participacion?: number;
}

interface Match {
  id: number;
  local: string;
  visitante: string;
  fecha: string;
  estado: 'upcoming' | 'live' | 'finished';
  goles_local: number | null;
  goles_visitante: number | null;
  fase: string;
  grupo?: string;
}
```

### 3. Sin duplicación (AHA > DRY) — RIESGO MEDIO

**Duplicación detectada:**
- `ensureNotificationsTables()` — 3 copias idénticas
- `CREATE TABLE IF NOT EXISTS user_payments` — 2 routes
- `ensureAuditLogsTable()` inline en `predictions/route.ts`
- El patrón `getSessionUser() → if (!user) return 401` repetido en cada route sin un wrapper

**Propuesta — HOF para auth guard:**
```typescript
// src/lib/withAuth.ts
type Handler = (req: NextRequest, user: UserSession) => Promise<NextResponse>;

export function withAuth(handler: Handler, requiredRole?: string) {
  return async (req: NextRequest): Promise<NextResponse> => {
    const user = await getSessionUser();
    if (!user) return NextResponse.json({ error: 'No autorizado' }, { status: 401 });
    if (requiredRole && user.tipo !== requiredRole && user.tipo !== 'superadmin') {
      return NextResponse.json({ error: 'No autorizado' }, { status: 403 });
    }
    return handler(req, user);
  };
}

// Uso:
export const GET = withAuth(async (req, user) => {
  // user guaranteed non-null here
});
```

### 4. Mínimos elementos (YAGNI) — RIESGO BAJO

- `ensureAuditLogsTable` definida como función inline dentro del handler POST de predictions es innecesariamente compleja. La tabla debería existir desde `init.sql`.
- El `cleanupCounter` en `proxy.ts` para limpiar el Map cada 500 requests es ingeniería ingeniosa pero innecesaria — un simple `setInterval` de limpieza sería más predecible.

---

## PILAR 3 — DDD y Lenguaje Ubicuo

### Diagnóstico Actual

**✅ Vocabulario consistente:**
- `prediction` (endpoints, DB, UI) — no mezcla con "apuesta/bet"
- `match` (endpoints y DB) — `local/visitante` son términos válidos del dominio fútbol
- `leaderboard` (endpoint, DB, UI) — consistente
- `company/companies` — consistente
- Roles: `externo/interno/admin/superadmin` — consistentes en todas las capas

**❌ Inconsistencias:**
- `tipo` para el rol del usuario → en la UI aparece como "rol", en la DB como `tipo`. El contrato del JWT lleva `tipo` → dificulta onboarding
- `tincaso` — campo completamente opaco. ¿Qué significa en el dominio de negocio?
- `logSystem` importado desde `mail` → el ubiquitous language no mapea "logging" a "mail"

### Nivel de Riesgo: BAJO

---

## PILAR 4 — Seguridad OWASP

### SEC-01: Information Leakage en Errores — RIESGO ALTO

**Diagnóstico:** 8+ rutas exponen `error.message` directamente en producción:

```
users/route.ts:33       → { error: error.message }
passkeys/route.ts:22,42 → { error: error.message }
notifications/route.ts:175,223,262 → { error: 'Error del servidor: ' + error.message }
profile/route.ts:126,152 → { error: `Error del servidor: ${error.message}` }
settings/route.ts:52,173 → { error: error.message }
companies/route.ts:110  → { error: 'Error del servidor: ' + error.message }
sync/route.ts:40,54     → { error: 'Error interno: ' + error.message }
```

Esto expone: nombres de tablas, columnas, queries SQL fallidas, mensajes de errores internos de PostgreSQL, rutas de archivo del servidor.

**Propuesta:**
```typescript
// PATRÓN CORRECTO — aplicar en todos los catch blocks
catch (error: any) {
  console.error('[route-name] Error:', error); // Log completo solo en servidor
  return NextResponse.json(
    { error: 'Error del servidor' }, // Solo mensaje genérico al cliente
    { status: 500 }
  );
}
```

### SEC-02: CSP con `unsafe-inline` en producción — RIESGO ALTO

**Diagnóstico:** `proxy.ts` línea 53:
```typescript
const scriptSrc = isProd
  ? "script-src 'self' 'unsafe-inline' https://static.cloudflareinsights.com"  // ← PROBLEMA
  : "script-src 'self' 'unsafe-inline' 'unsafe-eval' ...";
```

`unsafe-inline` en producción invalida completamente la protección XSS del CSP. Un atacante que logre inyectar un `<script>` inline puede ejecutar código sin restricción.

La causa raíz es `dangerouslySetInnerHTML` en `layout.tsx` para el service worker — que se puede reemplazar por un nonce.

**Propuesta:**
```typescript
// proxy.ts — generar nonce por request
const nonce = Buffer.from(crypto.randomUUID()).toString('base64');

const scriptSrc = isProd
  ? `script-src 'self' 'nonce-${nonce}' https://static.cloudflareinsights.com`
  : `script-src 'self' 'nonce-${nonce}' 'unsafe-eval' https://static.cloudflareinsights.com`;

res.headers.set('x-nonce', nonce); // pasar al layout via header

// layout.tsx — usar nonce en lugar de dangerouslySetInnerHTML
// Next.js 13+ permite leer el nonce del header en Server Components
import { headers } from 'next/headers';
const nonce = (await headers()).get('x-nonce') ?? '';

<script nonce={nonce}>{`
  if ('serviceWorker' in navigator) { ... }
`}</script>
```

### SEC-03: IDOR en predictions GET con `?matchId=X` — RIESGO MEDIO

**Diagnóstico:** `GET /api/predictions?matchId=123` retorna predicciones de TODOS los usuarios para ese partido, incluyendo nombres y avatares:

```sql
SELECT p.*, u.nombre, u.avatar, u.tipo
FROM predictions p
JOIN users u ON p.user_id = u.id
WHERE p.match_id = $1
ORDER BY p.puntos DESC, u.nombre ASC
```

Si esto es intencional (mostrar ranking del partido), está bien documentado. Si no, es una exposición de datos.  
**Acción:** Confirmar intencionalidad y documentar en el endpoint con un comentario explícito.

### SEC-04: Rate Limiter in-memory no efectivo en cluster — RIESGO MEDIO

**Diagnóstico:** `proxy.ts` usa un `Map<string, RateLimitEntry>` en memoria. Con Podman corriendo `app_1` y `app_2` como procesos separados, cada uno tiene su propio store. Un atacante puede enviar el doble de requests permitidos rotando entre workers.

**Propuesta a mediano plazo:**
```typescript
// Reemplazar Map por Redis con atomic increment
import { createClient } from 'redis';
const redis = createClient({ url: process.env.REDIS_URL });

async function rateLimit(key: string, limit: number, windowMs: number): Promise<boolean> {
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, Math.ceil(windowMs / 1000));
  return count <= limit;
}
```

### SEC-05: `dangerouslySetInnerHTML` en layout.tsx — RIESGO BAJO (Aceptado)

El único uso es para el registro del service worker con código completamente estático. No hay interpolación de datos de usuario. RIESGO ACEPTADO hasta implementar nonce (SEC-02).

---

## PILAR 5 — Estrategia de Despliegue

### Diagnóstico Actual

**✅ DECISIÓN CORRECTA: Monolito Modular**
La elección de un monolito Next.js bien modularizado es la arquitectura correcta para el equipo actual. Los microservicios agregarían: overhead de red, serialización, service discovery, circuit breakers — complejidad injustificada para este contexto.

**✅ Podman cluster bien pensado:**
- `nginx-lb → app_1/app_2 → postgres` — separación de responsabilidades clara
- `standalone` build para distribución en contenedores
- Health endpoint en `/api/health`

**⚠️ Limitaciones conocidas del modelo actual:**

1. **SSE cross-worker**: Los eventos SSE no cruzan workers PM2/Podman. El código tiene el workaround correcto (polling cada 90s) pero es una limitación arquitectural a documentar.

2. **Rate limiter in-memory**: Ver SEC-04. Solución: Redis o un reverse proxy con rate limiting (nginx tiene `limit_req`).

3. **Estado del leaderboard**: `recalculate_leaderboard()` se llama sincrónicamente en cada predicción → puede ser cuello de botella con 100+ usuarios simultáneos.

### Recomendación de Evolución

```
Hoy → Mediano plazo → Largo plazo
Monolito Next.js → + Redis (caché + rate limit) → Separar API pública si hay >500 usuarios concurrentes
```

---

## Resumen Ejecutivo

| Pilar | Calificación | Issues Críticos |
|-------|-------------|-----------------|
| 1. Arquitectura | 7/10 | logSystem en mail.ts, DDL en runtime |
| 2. Diseño Simple | 5/10 | Zero tests, any[] proliferado |
| 3. DDD | 8/10 | `tincaso` opaco, `tipo` ambiguo |
| 4. Seguridad | 6/10 | error.message expuesto, CSP unsafe-inline |
| 5. Despliegue | 7/10 | Rate limiter inefectivo en cluster |

**Score Global: 6.6/10 — Maduro pero con deuda técnica de seguridad activa**

### Priorización de Fixes

| Prioridad | ID | Fix | Esfuerzo |
|-----------|-----|-----|---------|
| 🔴 CRÍTICO | SEC-01 | Eliminar error.message de responses | 2h |
| 🔴 CRÍTICO | SEC-02 | CSP nonce en lugar de unsafe-inline | 4h |
| 🟡 ALTO | DES-01 | Agregar vitest + tests para lib/validation | 6h |
| 🟡 ALTO | ARC-01 | Mover logSystem a src/lib/logger.ts | 1h |
| 🟡 ALTO | ARC-02 | Mover DDL a init.sql o migrations.ts | 3h |
| 🟠 MEDIO | DES-02 | Tipos explícitos en AppContext (no any[]) | 4h |
| 🟠 MEDIO | SEC-03 | Documentar/proteger predictions GET matchId | 1h |
| 🔵 BAJO | DEP-01 | Redis para rate limiter cross-worker | 8h |

**Esfuerzo total estimado: ~29 horas**
