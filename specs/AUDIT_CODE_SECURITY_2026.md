# Especificación: Auditoría de Seguridad y Calidad - ElitePass Mundial

**ID:** AUDIT-SECURITY-2026-Q2  
**Estado:** 🔴 CRÍTICO - EN ACCIÓN  
**Fecha Creación:** 2026-06-12  
**Auditor:** Claude Code  
**Reviewed By:** (Pending Senior Dev)

---

## 1. Descripción General

Auditoría completa de seguridad, vulnerabilidades CVE, calidad de código y arquitectura del proyecto **ElitePass Mundial** (Next.js 16 + PostgreSQL + JWT Auth).

**Resultado:** 3 críticas, 3 altas, 208 quality issues identificados.

---

## 2. Hallazgos Críticos

### 2.1 Secretos Expuestos
```
Severidad: CRÍTICA
Archivos: .env.local (repo)
Impacto: Total compromise
```

**Secretos comprometidos:**
- DB_PASSWORD
- JWT_SECRET  
- SYNC_SECRET
- FOOTBALL_API_KEY
- MAIL_GRAPH_CLIENT_SECRET
- VAPID_PRIVATE_KEY
- CF_API_TOKEN
- AZURE_STORAGE_CONNECTION_STRING

**Remediación:**
1. Rotar TODOS los secretos en producción hoy
2. Ejecutar: `git filter-branch --force --tree-filter 'rm -f .env.local' -- --all`
3. Redeployment coordinado

**Referencia:** [[fix_secretos_expuestos]]

---

### 2.2 JWT Session Timeout = 12 horas
```
Severidad: CRÍTICA
Archivo: /src/lib/auth.ts línea 18
Código: const SESSION_TTL_SECONDS = 60 * 60 * 12
```

**Problema:**
- Token robado = 12 horas de acceso
- Estándar OWASP: máx 30 min para tokens
- Sin refresh token mechanism

**Remediación:**
- Cambiar a 2 horas
- Implementar refresh token (7 días)
- Coordinar con `/src/lib/identity-sync.ts`

**Bloqueo:** ⚠️ Requiere coordinación identity antes de implementar

**Referencia:** [[fix_jwt_ttl_reduction]]

---

### 2.3 Content Security Policy - Unsafe Scripts
```
Severidad: CRÍTICA  
Archivo: /src/proxy.ts línea 53
```

**Actual:**
```
'script-src 'self' 'unsafe-inline' 'unsafe-eval'
```

**Riesgo:** XSS → token theft, session hijacking, malware injection

**Remediación:**
- Remover `'unsafe-inline'` y `'unsafe-eval'`
- Usar nonces para scripts inline
- Testing cross-browser

**Referencia:** [[fix_csp_hardening]]

---

## 3. Vulnerabilidades Altas

### 3.1 Open Redirect
**Ubicación:** `/src/app/api/auth/identity-callback/route.ts:154-156`  
**Código:** `if (redirectTo.startsWith('http')) { return redirectTo; }`  
**Fix:** Whitelist de rutas permitidas + rechazar URLs externas

### 3.2 Auth Callback XSS
**Ubicación:** `/src/app/api/auth/identity-callback/route.ts:160-197`  
**Problema:** Token sin JSON.stringify()  
**Fix:** Usar `JSON.stringify()` para escapar valores

### 3.3 /api/sync Query String Auth
**Ubicación:** `/src/app/api/sync/route.ts`  
**Problema:** Secreto en URL visible en logs  
**Fix:** Bearer token + timing-safe comparison

---

## 4. Problemas de Calidad

### 4.1 TypeScript: 181 `any` Types
- 49 archivos afectados
- Crítico: `/src/app/(app)/admin/page.tsx` (27 any)
- Estimado fix: 15-20 horas

### 4.2 Componente Monolítico
- `admin/page.tsx`: 2,432 líneas
- ~500 useState variables  
- Necesita: Dividir en 6 componentes

### 4.3 13 React Hooks Violations
- Missing dependency arrays
- setState en effects
- Memory leaks potenciales

### 4.4 Rate Limiting Débil
- `/api/auth`: 10 req/min (debe ser 2-3)
- `/api/register`: 5 req/min (debe ser 2-3)
- Vulnerable a brute force

### 4.5 Validación de Archivos
- No valida MIME type real
- No valida tamaño
- DoS por archivos grandes posible

---

## 5. Plan de Remediación

### Fase 1: Críticos (Hoy)
- [ ] Rotar secretos en prod (2h)
- [ ] Documentar cambios identity-sync necesarios (1h)

### Fase 2: Coordinación Identity (1-2 días)
- [ ] Reunión team identity (1h)
- [ ] Cambios JWT TTL (2h)
- [ ] Cambios auth callback (2h)
- [ ] Testing (1h)

### Fase 3: Security Hardening (3-4 días)
- [ ] CSP hardening (3h)
- [ ] /api/sync fix (1.5h)
- [ ] Rate limiting (1h)
- [ ] File validation (1.5h)

### Fase 4: Code Quality (1-2 semanas)
- [ ] Crear tipos base (3h)
- [ ] Migrar any types (15-20h)
- [ ] Dividir admin/page.tsx (8-10h)
- [ ] Custom hooks (4h)

**Estimado Total:** 50 horas (~1 semana full-time)

---

## 6. Criterios de Aceptación

### Por Vulnerabilidad Crítica
- [ ] Código fix implementado
- [ ] Tests pasan (sin nuevas vulnerabilidades)
- [ ] Code review aprobado (2+ seniors)
- [ ] Deployed a staging
- [ ] QA verification
- [ ] Deployed a producción

### Por Quality Issue
- [ ] Tipos definidos correctamente
- [ ] ESLint pasa sin warnings
- [ ] Tests pasan
- [ ] No performance regression
- [ ] Code review aprobado

---

## 7. Riesgos y Mitigación

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|--------|-----------|
| Secretos aún en git | ALTA | CRÍTICO | Filter-branch hoy |
| JWT theft en production | ALTA | CRÍTICO | Reducir TTL ASAP |
| XSS exploitation | MEDIA | ALTO | Fix auth callback + CSP |
| Identity sync break | MEDIA | ALTO | Coordinación antes de deploy |
| Incompatibilidad browser | BAJA | MEDIO | Testing cross-browser CSP |

---

## 8. Documentación y Referencias

- **Reporte Ejecutivo:** `/tmp/AUDIT_REPORT_SENIOR_DEV.md`
- **Análisis Detallado:** `/tmp/QUALITY_ANALYSIS.md`
- **Quick Fixes:** `/tmp/QUICK_FIXES.md`
- **Memory:** [[code_audit_findings_2026_06_12]]
- **Skill:** [[skill_code_auditor]]

---

## 9. Siguientes Pasos

1. ✅ Distribuir reporte a Senior Dev
2. ⏳ Senior Dev revisa y aprueba plan
3. ⏳ Crear GitHub Issues para cada fix
4. ⏳ Asignar tasks a developers
5. ⏳ Comenzar Fase 1 (críticos)

---

## 10. Definición de Hecho

```gherkin
Given: Auditoría completada
When: Todos los fixes implementados
Then: 
  - Cero secretos en repo
  - Cero vulnerabilidades críticas/altas
  - ESLint pasa con <5 warnings
  - 100% de tests pasando
  - JWT TTL ≤ 2 horas
  - CSP sin unsafe-*
  - Rate limiting en <3 req/min
  - Archivos validados correctamente
  - Admin/page.tsx dividido <700 líneas
  - any types <10 en codebase
  - Segunda auditoría pasa con "BAJA SEVERIDAD"
```

---

**Auditor:** Claude Code  
**Fecha:** 2026-06-12  
**Próxima Auditoría:** 2026-07-12  
**Estado:** 🔴 CRÍTICO REQUERIDO IMPLEMENTAR
