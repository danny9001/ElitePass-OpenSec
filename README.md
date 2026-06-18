# ElitePass OpenSec — Especificaciones & Documentación Centralizada

Repositorio centralizado de **especificaciones de diseño**, **documentación de arquitectura**, **planes de implementación** y **auditorías de seguridad** para todos los proyectos de la plataforma **ElitePass** (Mundial 2026, Identity, Reservas, POS, Payments, Telegram, Monitor).

**Sincronización:** Este repo se sincroniza automáticamente con sistemas de Antigravity y ambos servidores de producción.

---

## 📁 Estructura

### `/specs/antigravity/`
Especificaciones maestras de cada módulo:
- `elitepass-mundial-master-2026.md` — Documento maestro centralizado
- `elitepass-mundial.md` — Especificaciones funcionales del mundial
- `elitepass-mundial-architecture.md` — Arquitectura técnica y deployment
- `elitepass-mundial-responsive-design.md` — Responsive design (mobile/desktop)
- `elitepass-identity.md` — Sistema de autenticación SSO
- `elitepass-reservas.md` — Reserva de espacios y gestión de grupos
- `elitepass-pos.md` — Sistema de punto de venta
- `elitepass-payments.md` — Pasarelas de pago
- `elitepass-noti-telegram.md` — Notificaciones vía Telegram
- `elitepass-monitor.md` — Monitoreo y alertas
- `MEMORY.md` — Índice de memorias y contexto del proyecto

### `/specs/antigravity/` — Auditorías & Verificaciones
- `project_code_audit_findings.md` — Auditoría de seguridad 2026-06-12
- `implementation_progress_2026.md` — Progreso de fixes de seguridad
- `audit_implementation_plan_2026.md` — Plan de implementación 50h
- `verification_score_notifications_2026.md` — **[NUEVO]** Verificación de timing (2026-06-18)

### `/changes/`
Historial de cambios, propuestas y tareas (45+ directorios):
```
changes/
├── worldwide-responsive-design-2026/
│   ├── proposal.md      # Justificación del cambio
│   ├── design.md        # Análisis técnico
│   └── tasks.md         # Checklist de implementación
├── security-audit-passwords/
├── identity-deploy/
└── ... (múltiples cambios documentados)
```

### `/specs/` — Especificaciones Técnicas
- `apuestas-mundial/spec.md` — Spec funcional del mundial
- `auth/spec.md` — Autenticación
- `payments/spec.md` — Pagos
- `security/spec.md` — Seguridad
- `telegram-notifications/spec.md` — Notificaciones
- `wallet-credentials/spec.md` — Wallet

---

## 🔄 Sincronización

### Local ↔ Remote
Este repositorio se sincroniza automáticamente:

**Desde:** `/home/soporte/openspec/` (servidor local)  
**Hacia:** `git@github.com:danny9001/ElitePass-OpenSec.git`  
**Servidores Finales:**
- 🖥️ **Servidor 1 (10.0.0.4)** — Producción principal
- 🖥️ **Servidor 2** — Replica/Standby

```bash
# Clone en nuevo servidor:
git clone https://github.com/danny9001/ElitePass-OpenSec.git
cd ElitePass-OpenSec
```

---

## 📋 Documentos Recientes

### Verificación 2026-06-18 — Score Notifications & Betting Closure ✅

**Hallazgos Clave:**
- ✅ Entrada de scores: Sin restricción (admin-only)
- ✅ Notificación 1:30 antes: Ventana 75-105 min confirmada
- ✅ Notificación 1 hora antes: Ventana 45-75 min confirmada
- ✅ Apuestas cierran: 15 min antes del kickoff validado

**Evidencia:** Ver `specs/antigravity/verification_score_notifications_2026.md`

### Auditoría de Seguridad 2026-06-12

**Vulnerabilidades Identificadas:** 3 críticas, 10 altas, 13 medias  
**Plan de Remediación:** 50 horas, 4 fases, 10 developers  
**Status:** ✅ Fase 1-2 completadas

**Artefactos:**
- `project_code_audit_findings.md` — Detalle de vulnerabilidades
- `audit_implementation_plan_2026.md` — Plan de fixes
- `implementation_progress_2026.md` — Seguimiento de implementación

---

## 🎯 Proyectos Incluidos

| Proyecto | Descripción | Estado |
|----------|-------------|--------|
| **elitepass-mundial** | Plataforma de pronósticos Mundial 2026 | ✅ Producción |
| **elitepass-identity** | Sistema SSO centralizado | ✅ Producción |
| **elitepass-reservas** | Reserva de espacios y grupos | ✅ Producción |
| **elitepass-pos** | Sistema de punto de venta | ✅ Producción |
| **elitepass-payments** | Pasarelas de pago (MercadoPago, Stripe, etc) | ✅ Producción |
| **elitepass-noti-telegram** | Bot de notificaciones Telegram | ✅ Producción |
| **elitepass-monitor** | Monitoreo infrastructure (Zabbix + ChatOps) | ✅ Producción |

---

## 🔐 Seguridad & Compliance

### Estándares Aplicados
- **OWASP Top 10** — Validación de entrada, XSS prevention, CSRF tokens
- **JWT Security** — TTL corto (12h), httpOnly cookies, rotation automática
- **Database** — Prepared statements, role-based access control (RBAC)
- **Transport** — TLS 1.3+, HSTS, CSP headers
- **Secretos** — Gestión via .env, nunca en git

### Proceso de Auditoría Continua
1. Auditoría de código automatizada (quarterly)
2. Revisión de seguridad manual (post-deployment)
3. Penetration testing anual
4. Compliance checks (GDPR, LGPD)

---

## 📞 Contacto & Soporte

- **Lead Técnico:** Danny (danny9001@gmail.com)
- **DevOps:** Genial IT
- **Documentación:** Revisada 2026-06-18
- **Última Sincronización:** 2026-06-18 03:30 UTC

---

## 📝 Licencia

Privado — Genial IT SPA · 2026

**Para cambios en este repositorio:**
1. Crear branch: `git checkout -b feature/descripcion`
2. Actualizar correspondiente en `/home/soporte/openspec/` en servidor local
3. Hacer commit con co-author: `Claude Haiku 4.5`
4. Push a `github.com:danny9001/ElitePass-OpenSec.git`
5. Sincronización automática a ambos servidores

---

**Última actualización:** 2026-06-18 03:35 UTC  
**Commit:** Sync memory to OpenSpec
