---
name: code_audit_findings_2026_06_12
description: Hallazgos de auditoría completa de seguridad y calidad en elitepass-mundial
metadata: 
  node_type: memory
  type: project
  date: 2026-06-12
  status: active
  originSessionId: fe4a7f78-cb44-4fe0-a950-02411d3195c1
---

# Auditoría de Código - ElitePass Mundial

## Estado Actual
**Riesgo Crítico:** 3 vulnerabilidades críticas + 208 errores de calidad
**Acción Requerida:** Coordinación de fixes de identity/auth antes de implementar

## 🔴 CRÍTICAS (Acción Inmediata)

### 1. Secretos Expuestos en .env.local
- **Severidad:** CRÍTICA
- **Archivos:** `.env.local` en repositorio
- **Secretos comprometidos:** DB_PASSWORD, JWT_SECRET, SYNC_SECRET, FOOTBALL_API_KEY, MAIL_GRAPH_CLIENT_SECRET, VAPID_PRIVATE_KEY, CF_API_TOKEN, AZURE_STORAGE_CONNECTION_STRING
- **Acción:** Rotar TODOS los secretos en producción (10.0.0.4:5001) hoy
- **Remedición:** `git filter-branch --force --tree-filter 'rm -f .env.local' -- --all`
- **Costo:** 1 hora (coordinación + rotación)

### 2. JWT TTL = 12 horas (MUY ALTO)
- **Ubicación:** `/src/lib/auth.ts` línea 18
- **Problema:** `const SESSION_TTL_SECONDS = 60 * 60 * 12`
- **Riesgo:** Token robado = 12 horas de acceso no autorizado
- **Fix:** Cambiar a 2 horas + implementar refresh tokens (15-30 min access)
- **Costo:** 2 horas (coordinar con identity-sync.ts)
- **Bloqueante:** SÍ - afecta sesiones activas

### 3. CSP Permite 'unsafe-eval' y 'unsafe-inline'
- **Ubicación:** `/src/proxy.ts` línea 53
- **Actual:** `'script-src 'self' 'unsafe-inline' 'unsafe-eval'`
- **Riesgo:** XSS puede ejecutarse libremente → robo de cookies/tokens
- **Fix:** Remover unsafe-*, usar nonces para scripts inline
- **Costo:** 3 horas (testing en todos los browsers)

## 🟠 ALTAS (Próxima Sprint)

### 4. Autenticación por Query String en /api/sync
- **Ubicación:** `/src/app/api/sync/route.ts`
- **Problema:** Secreto en URL sin encriptación
- **Visible en:** logs, historial navegador, caché HTTP
- **Fix:** Authorization header + Bearer token + timing-safe comparison
- **Costo:** 1.5 horas

### 5. XSS en Auth Callback
- **Ubicación:** `/src/app/api/auth/identity-callback/route.ts` líneas 160-197
- **Problema:** Token y redirectUrl sin escapar en HTML: `token: '${sessionToken}'`
- **Fix:** `JSON.stringify()` + validar redirectUrl contra whitelist
- **Costo:** 1 hora
- **Bloqueante:** SÍ - identity callback

### 6. Open Redirect Vulnerability
- **Ubicación:** `/src/app/api/auth/identity-callback/route.ts` líneas 154-156
- **Código:** `if (redirectTo.startsWith('http')) { return redirectTo; }`
- **Riesgo:** Phishing, redirección a sitios maliciosos
- **Fix:** Whitelist de rutas permitidas, rechazar URLs con `http` o `//`
- **Costo:** 1 hora

## 🟡 CALIDAD (1-2 semanas)

### 7. 181 `any` types (49 archivos)
- **Crítico:** `/src/app/(app)/admin/page.tsx` (27 any types)
- **Impacto:** Pérdida total de type safety
- **Costo:** 15-20 horas
- **Plan:** Crear tipos base → migrar gradualmente

### 8. admin/page.tsx Monolítico (2,432 líneas)
- **Problema:** ~500 variables useState, imposible mantener
- **Solución:** Dividir en 6 componentes + custom hooks
- **Costo:** 8-10 horas
- **Priority:** P2 (después de fixes críticos)

### 9. 13 React Hooks Violations
- **Ubicaciones:** effects sin dependency arrays, setState en effects
- **Impacto:** Cascading renders, memory leaks
- **Costo:** 3-4 horas

### 10. Validación de Archivos Insuficiente
- **Ubicación:** `/src/app/api/admin/payments/route.ts` líneas 137-182
- **Problema:** Falta validación MIME, tamaño, contenido
- **Fix:** Whitelist tipos + máximo 10MB + validar contenido real
- **Costo:** 1.5 horas

## 📊 Estadísticas
- **Errores ESLint:** 208
- **Warnings ESLint:** 37
- **Archivos afectados:** 40+ de 71 (56%)
- **LOC auditadas:** 11,646
- **Estimado total fixes:** 43-50 horas (~1 semana full-time)

## 🎯 Plan de Acción Priorizado

### Week 1 (Críticos + Security)
1. Rotar secretos en producción (1h)
2. Remover .env.local del git (1h)
3. Reducir JWT TTL a 2h (2h) - **COORDINAR CON IDENTITY**
4. Fix Auth Callback XSS (1h) - **COORDINAR CON IDENTITY**
5. Agregar whitelist redirectUrl (1h)
6. Fix /api/sync authentication (1.5h)
7. Remover unsafe-eval de CSP (3h)
8. Fix validación de archivos pagos (1.5h)

**Subtotal:** 12.5 horas

### Week 2-3 (Quality + Performance)
1. Crear tipos base (admin.ts, AppContext)
2. Migrar `any` types → tipos específicos
3. Dividir admin/page.tsx
4. Crear custom hooks reutilizables
5. Optimizar <img> tags

**Subtotal:** 30-35 horas

## 🔗 Coordinación Requerida
- **Identity:** Cambios en JWT TTL, auth callback, redirectUrl validation
- **Producción:** Rotación de secretos, deployment de CSP changes
- **Testing:** Validar all flows con cambios auth/identity

## Documentos Generados
- `/tmp/CODE_QUALITY_INDEX.md` - Índice por rol
- `/tmp/QUALITY_SUMMARY.txt` - Ejecutivo (5 min)
- `/tmp/QUALITY_ANALYSIS.md` - Detallado (30 min)
- `/tmp/QUICK_FIXES.md` - Implementación paso a paso

## Estado Actual de Fixes
- ✅ Logo upload improvements (commit: a4e8764)
- ⏳ Aguardando coordinación de identity para fixes críticos
- 📋 Plan de implementación escalonado listo
