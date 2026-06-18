# ElitePass OpenSec

Repositorio centralizado de **especificaciones técnicas**, **documentación de arquitectura**, **planes de implementación** y **auditorías de seguridad** para todos los proyectos de la plataforma **ElitePass** de Genial IT.

Sincronizado entre VM00-RESERVAS-v2, VM02-MUNDIAL y los sistemas Antigravity y Codex.

---

## Estructura

### `/specs/antigravity/` — Especificaciones maestras

| Archivo | Contenido |
|---------|-----------|
| `elitepass-mundial-master-2026.md` | Documento maestro centralizado Mundial 2026 |
| `elitepass-mundial.md` | Especificaciones funcionales |
| `elitepass-mundial-architecture.md` | Arquitectura técnica y deployment |
| `elitepass-mundial-responsive-design.md` | Responsive design mobile/desktop |
| `elitepass-identity.md` | SSO/IAM centralizado |
| `elitepass-reservas.md` | Sistema de reservas y grupos |
| `elitepass-pos.md` | Punto de venta |
| `elitepass-payments.md` | Pasarelas de pago |
| `elitepass-noti-telegram.md` | Notificaciones Telegram |
| `elitepass-monitor.md` | Monitoreo e infraestructura |
| `MEMORY.md` | Índice de memorias y contexto del proyecto |

#### Auditorías

| Archivo | Descripción |
|---------|-------------|
| `AUDIT_CODE_SECURITY_2026.md` | Auditoría de seguridad 2026-06-12 |
| `audit_implementation_plan_2026.md` | Plan de remediación 50h, 4 fases |
| `implementation_progress_2026.md` | Seguimiento de fixes |
| `verification_score_notifications_2026.md` | Verificación timing notificaciones 2026-06-18 |

### `/specs/` — Especificaciones técnicas por dominio

```
specs/
├── apuestas-mundial/spec.md
├── auth/spec.md
├── backups/
├── payments/spec.md
├── security/spec.md
├── telegram-notifications/spec.md
└── wallet-credentials/spec.md
```

### `/changes/` — Historial de cambios (45+ entradas)

Cada directorio representa un cambio o fix con el patrón:

```
changes/<nombre>/
├── proposal.md   — Justificación del cambio
├── design.md     — Análisis técnico
└── tasks.md      — Checklist de implementación
```

Ejemplos recientes: `geoblock-failure-analysis`, `identity-branding-fixes`, `security-audit-passwords`, `skills-mundial-sync`.

---

## Proyectos documentados

| Proyecto | Servidor | URL | Estado |
|----------|----------|-----|--------|
| elitepass-reservas | VM00 | reservas.genial-it.net | Producción |
| elitepass-pos | VM00 | pos.genial-it.net | Producción |
| elitepass-payments | VM00 | — (port 3100) | Producción |
| elitepass-noti-telegram | VM00 | — (port 3200) | Producción |
| elitepass-identity | VM00 | id.genial-it.net | Producción |
| elitepass-monitor | VM00 | — (bot Telegram) | Producción |
| elitepass-mundial | VM02 | mundial.genial-it.net | Producción |

---

## Infraestructura

| Servidor | IP privada | OS | Rol |
|----------|-----------|-----|-----|
| VM00-RESERVAS-v2 | 10.0.0.5 | Ubuntu 24.04 ARM64 | Principal |
| VM02-MUNDIAL | 10.0.0.7 | Ubuntu 24.04 x86_64 | Mundial 2026 |

---

## Auditoría de seguridad 2026-06-12

**Vulnerabilidades:** 3 críticas · 10 altas · 13 medias  
**Plan:** 50 horas · 4 fases — Fases 1 y 2 completadas.

Ver `specs/AUDIT_CODE_SECURITY_2026.md` y `specs/antigravity/audit_implementation_plan_2026.md`.

---

## Convención para nuevos cambios

```bash
mkdir changes/<nombre>
touch changes/<nombre>/proposal.md
touch changes/<nombre>/design.md
touch changes/<nombre>/tasks.md
git add changes/<nombre>/
git commit -m "docs(<nombre>): agregar propuesta y diseño"
git push origin main
```

---

## Licencia

**LICENCIA CREATIVA ELITEPASS**
© 2024-2026 Genial IT — Todos los derechos reservados.

| Rol | |
|-----|--|
| Concepcion y direccion | Daniel Landivar |
| Asistencia en desarrollo | Claude · Antigravity · Codex |

Claude, Antigravity y Codex son herramientas de la linea de desarrollo de Genial IT.

El codigo, diseno y arquitectura son propiedad exclusiva de Genial IT. No se permite su uso, copia, modificacion ni distribucion sin autorizacion expresa y por escrito.

Hecho con humanos + IA · Genial IT · Bolivia
