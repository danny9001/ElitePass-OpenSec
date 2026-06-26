# Memory Index — ElitePass Mundial

**Última actualización:** 2026-06-19 UTC  
**Sincronización:** ✅ OpenSpec + Antigravity  
**Documento Maestro:** `/home/soporte/openspec/specs/antigravity/elitepass-mundial-master-2026.md`

## Skills
- [Code Auditor Skill](../skills/skill-code-auditor.md) — Auditoría de seguridad, calidad y vulnerabilidades
- [Skill para modificaciones](feedback_skill_for_modifications.md) — Siempre usar el Skill tool como referencia antes de cualquier modificación
- [Implementation Coordinator Skill](../skills/skill-implementation-coordinator.md) — Coordinación paralela de auditorías + implementación autónoma (NUEVA)
- [Mundial Stats & Logs Skill](skill-elitepass-mundial-stats.md) — Sistema de logging de eventos (logSystem, categorías, audit_logs) y dashboard StatsTab

## Feedback & Estándares
- [Versión en cada commit](feedback_version_on_commit.md) — Actualizar package.json a 1.1.XXX en cada commit, XXX = número secuencial del commit
- [Tablas en mobile = tarjetas](feedback_mobile_no_hscroll.md) — Tablas con `hidden sm:block` + tarjetas con `sm:hidden`, sin scroll horizontal en mobile

## Infraestructura
- [Seguridad del servidor](project_server_security.md) — Puerto 22 debe estar cerrado; SSH va por puerto 5001 o 5011
- [Arquitectura de producción](project_architecture.md) — PM2 bare-metal cluster (4 procesos puerto 3002), nginx proxy (443→3002)

## Bugs Conocidos & Fixes
- [Fix: Session SameSite](fix_session_sameSite.md) — sameSite: 'strict' rompía login, cambiar a 'lax'

## Verificaciones & Testing
- [Verificación: Scores, Notificaciones, Cierre de Apuestas 2026-06-18](verification_score_notifications_2026.md) — ✅ PASS — Sistema funcionando correctamente, notificaciones a 1:30 y 1 hora, apuestas cierran 15 min antes

## Auditorías & Implementación
- [Auditoría de Código 2026-06-12](project_code_audit_findings.md) — 3 críticos, 10 altos, 13 medios; plan de 50h; secretos expuestos, JWT TTL=12h, CSP unsafe
- [Plan de Implementación 2026-06-12](audit_implementation_plan_2026.md) — 14 vulnerabilidades, 4 fases, 10 developers, 50h, status: IMPLEMENTACIÓN EN PROGRESO
- [Progreso de Implementación 2026-06-12](implementation_progress_2026.md) — Fase 1-2 COMPLETADAS ✅ (JWT TTL, Open Redirect, XSS, Bearer tokens, file validation); Fase 3 próxima (type safety)
- [Evaluación Arquitectónica 2026-06-19](project_arch_evaluation_2026.md) — Score 7.6/10 post-fix; SEC-01+SEC-02 CERRADOS ✅ v1.1.88; pendiente: DES-01 zero tests, ARC-01 logSystem, ARC-02 DDL runtime

## UI/UX
- [Responsive Design 2026-06-12](responsive_design_2026.md) — 6 páginas refactoradas, mobile compacto ↔ desktop 1920px desplegado ✅

---

## 🔄 Sincronización con OpenSpec & Antigravity

### Archivos Sincronizados

**OpenSpec Location:** `/home/soporte/openspec/specs/antigravity/`

- `elitepass-mundial-master-2026.md` ⭐ — Documento maestro centralizado
- `elitepass-mundial-responsive-design.md` — Responsive design completado
- `elitepass-mundial-architecture.md` — Arquitectura PM2 bare-metal
- `elitepass-mundial.md` — Spec general (existente)

### Procedimiento de Sincronización

1. **Actualizar memorias locales** → `/home/soporte/.claude/projects/-home-soporte/memory/`
2. **Copiar a OpenSpec** → `openspec/specs/antigravity/`
3. **Commit Git** → `git add . && git commit -m "Sync: Memorias 2026-06-12"`
4. **Antigravity detects** → Sincroniza con servidor remoto

### Herramientas de Sincronización

- **Antigravity CLI:** `/home/soporte/.gemini/antigravity-cli/`
- **Token:** `/home/soporte/.gemini/antigravity-cli/antigravity-oauth-token`
- **OpenSpec:** `/home/soporte/openspec/`

### Comandos Sincronización

```bash
# 1. Copiar memorias a OpenSpec
cp memory/*.md openspec/specs/antigravity/

# 2. Revisar cambios
cd openspec && git status

# 3. Commit
git add . && git commit -m "Sync: Memorias Claude 2026-06-12"

# 4. Push (Antigravity se sincroniza automáticamente)
git push origin main
```
