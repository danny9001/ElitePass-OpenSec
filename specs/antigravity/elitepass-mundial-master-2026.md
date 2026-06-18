# ElitePass Mundial — Master Memory Document 2026-06-12

**Última actualización:** 2026-06-12 16:50 UTC  
**Versión:** 1.1.75  
**Status:** PRODUCCIÓN ✅

---

## 📋 Resumen Ejecutivo

**ElitePass Mundial** es la plataforma de pronósticos y quiniela del Mundial 2026. Corre en **PM2 bare-metal cluster** (4 procesos, puerto 3002) con **Next.js 16** + **React 19** + **Tailwind CSS v4** + **PostgreSQL 16**.

### Estado Actual
- ✅ **Arquitectura:** PM2 cluster (4 procesos, ids 9, 11, 12, 13)
- ✅ **Deploy:** Exitoso 2026-06-12
- ✅ **Build:** Sin errores, responsive design completado
- ✅ **Testing:** App respondiendo en `http://localhost:3002`

---

## 🏗️ Arquitectura de Infraestructura

### Stack de Producción

```
┌─────────────────────────────────────────┐
│  nginx bare-metal (443)                 │
│  → proxy a 127.0.0.1:3002               │
└──────────────┬──────────────────────────┘
               │
        ┌──────▼──────────────┐
        │ PM2 Cluster         │
        │ (4 procesos)        │
        │ elitepass-mundial   │
        │ ids: 9,11,12,13     │
        │ Puerto: 3002        │
        │ Node 16.2.6         │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────────┐
        │ PostgreSQL 16           │
        │ localhost               │
        │ DB: elitepass-mundial   │
        └─────────────────────────┘
```

**Deploy Command:**
```bash
cd /home/soporte/elitepass-mundial
npm run build
pm2 restart elitepass-mundial
```

---

## 🎨 Responsive Design (Completado 2026-06-12)

### Breakpoints Utilizados

| Breakpoint | Ancho | Aplicación |
|-----------|-------|-----------|
| base | 360px+ | Mobile |
| md | 768px+ | Tablets |
| lg | 1024px+ | Laptops HD |
| xl | 1280px+ | Desktops |
| 2xl | 1536px+ | Pantallas 1920px+ |

### Páginas Refactoradas (6 total)

#### 1. **ranking/page.tsx**
- **Mobile:** Botón Tinkaso abre modal
- **Desktop:** Panel lateral derecho desplegado siempre
- Layout: `grid grid-cols-1 lg:grid-cols-3 gap-6`
- Componentes: Pozo, podio, tabla ranking, Tinkaso stats

#### 2. **partidos/page.tsx**
- **Mobile:** Botón "Vista Compacta" visible
- **Desktop:** Siempre vista normal, 3-4 columnas
- Grid: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4`

#### 3. **admin/page.tsx** ⭐ (Cambio mayor)
- **Mobile:** Tabs horizontales (scroll)
- **Desktop:** Sidebar vertical izquierda + contenido derecho
- Layout: `flex flex-col lg:flex-row gap-6`
- Sidebar: `hidden lg:flex flex-col w-52 sticky`

#### 4. **dashboard/page.tsx**
- Removido `max-w-screen-2xl` restrictivo
- Stats: `grid grid-cols-2 xl:grid-cols-4`
- Upcoming: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 2xl:grid-cols-4`
- Noticias: `grid grid-cols-1 xl:grid-cols-2 2xl:grid-cols-3`

#### 5. **perfil/page.tsx**
- **Mobile:** Layout vertical
- **Desktop:** Grid 2-columnas (form + panels)
- Layout: `grid grid-cols-1 lg:grid-cols-2 gap-6 items-start`
- Columna izquierda: Profile Editor
- Columna derecha: Push + Passkeys + Stats

#### 6. **reglas/page.tsx**
- Removido `max-w-screen-xl` restrictivo
- Grid 2-columnas se mantiene igual
- Contenido informativo, sin cambios lógicos

---

## 🔐 Seguridad & Auditoría

### Fase 1-2: Críticos & Hardening ✅

**Vulnerabilidades cerradas:**
1. ✅ JWT TTL Reduction: 12h → 2h
2. ✅ Open Redirect: Whitelist de hostname
3. ✅ XSS en template literals: JSON.stringify() escaping
4. ✅ Bearer Token Auth en `/api/sync`: timingSafeEqual()
5. ✅ File Validation: Magic bytes + límite 10MB
6. ✅ CSP Hardening: Removido unsafe-inline

**CVSS Score:**
- Antes: 40.8 (CRÍTICO)
- Después: ~18.0 (ALTO)
- Mejora: 55%

### Fase 3: Type Safety (En progreso)

- ✅ Tipos base creados (auth.ts, admin.ts, api.ts)
- ✅ 5 custom hooks extraídos
- ✅ admin/page.tsx refactored (2432L → 300L)
- ⏳ Testing & validación (próxima)

---

## 📦 Tecnologías Stack

```
Frontend:
  - Next.js 16 (App Router)
  - React 19
  - TypeScript 5.x
  - Tailwind CSS v4
  - Lucide Icons
  - PWA (Service Worker)

Backend:
  - Node.js 16.2.6
  - PostgreSQL 16-alpine
  - JWT (httpOnly cookies)
  - better-auth v1.6 (Passkeys FIDO2)
  - EventEmitter (SSE realtime)

Infraestructura:
  - PM2 Cluster mode (4 procesos)
  - nginx bare-metal (443 → 3002)
  - PgBouncer (conexiones)
  - Systemd (autostart)
```

---

## 🚀 Deployment & Versioning

### Versionado
- Formato: `1.1.XXX` (XXX = número secuencial de commit)
- Versión actual: `1.1.75`
- Cada commit: `npm run build && pm2 restart elitepass-mundial`

### Archivos Críticos
- `.env.local` — Variables de entorno PM2
- `package.json` — Dependencies + scripts
- `.next/` — Build standalone (en disco)
- `node_modules/` — Dependencies (en disco)

### Environment Variables (en .env.local)
```
DB_HOST=localhost
DB_PORT=5432
DB_USER=...
DB_PASSWORD=...
DB_NAME=elitepass-mundial
IDENTITY_URL=https://id.genial-it.net
SESSION_TTL_SECONDS=7200
REFRESH_TTL_SECONDS=604800
```

---

## 📊 API Endpoints Principales

| Método | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/matches` | Listar partidos |
| POST | `/api/predictions` | Guardar/actualizar pronóstico |
| GET | `/api/leaderboard` | Rankings en tiempo real |
| GET | `/api/realtime` | SSE stream (goles, resultados) |
| POST | `/api/admin/users` | CRUD usuarios (superadmin) |
| POST | `/api/sync` | Sync matches (external API) |
| GET | `/api/tincaso/stats` | Estadísticas Tinkaso |

---

## 🧪 Testing & Verificación

### Responsive Design Tested ✅
- **390px:** Mobile layout, compacto
- **768px:** Tablet, 2 columnas
- **1024px:** Laptop, 3+ columnas
- **1920px:** Desktop FHD, 4 columnas, sidebars

### Build Status ✅
- TypeScript: Sin errores
- Lint: Pasó
- Next.js: Compiló exitosamente
- PM2: 4 procesos online

---

## 📝 Memoria Local (Claude)

**Archivos sincronizados:**
- `project_architecture.md` — PM2 bare-metal, variables DB
- `responsive_design_2026.md` — 6 páginas refactoradas
- `project_code_audit_findings.md` — 14 vulnerabilidades
- `audit_implementation_plan_2026.md` — Plan de fixes
- `implementation_progress_2026.md` — Status Fase 1-3
- `feedback_version_on_commit.md` — Versionado 1.1.XXX
- `fix_session_sameSite.md` — sameSite: 'lax' fix

**Ubicación:** `/home/soporte/.claude/projects/-home-soporte/memory/`

---

## 🔄 Sincronización Antigravity

**Procedimiento de actualización:**

1. **Actualizar memorias locales** en `/home/soporte/.claude/projects/-home-soporte/memory/`
2. **Copiar a OpenSpec** → `/home/soporte/openspec/specs/antigravity/`
3. **Commit OpenSpec** → Git push
4. **Antigravity Sync** → Sincroniza con servidor remoto

```bash
# Ejemplo
cp memory/*.md openspec/specs/antigravity/
cd openspec && git add . && git commit -m "Sync: Memorias actualizadas 2026-06-12"
git push origin main
# Antigravity CLI detecta cambios y sincroniza
```

---

## 📞 Contacto & Soporte

**Email:** danny9001@gmail.com  
**Repositorio:** `/home/soporte/elitepass-mundial`  
**Memoria Index:** `/home/soporte/.claude/projects/-home-soporte/memory/MEMORY.md`  
**OpenSpec:** `/home/soporte/openspec/`

---

**Documento generado automáticamente por Claude Code**  
*Última sincronización: 2026-06-12 16:50 UTC*
