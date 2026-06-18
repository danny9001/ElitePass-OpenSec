# Diseño — Estandarización ElitePass 2026-06-05

## Arquitectura de conocimiento resultante

```
GitHub (skill-elite-pass-knowledge)   ← fuente de verdad de skills/reglas
    └── skill-elitepass-organizer.md      (reglas permanentes del ecosistema)
    └── skill-elitepass-server-monitor.md (deploy, git security, PM2)
    └── skill-elitepass-*.md              (skills especializados)

OpenSpec /changes/                    ← historial de decisiones y errores
    └── standardization-2026-06-05/       (este change)
    └── fix-payment-gateway-event-qr/     (completado)
    └── payment-methods-multi-tenant/     (completado)
    └── telegram-notification-service/    (completado → ahora elitepass-noti-telegram)

NotebookLM (Google, cuenta Premium)   ← análisis e inteligencia cruzada
    └── notebook: ElitePass-Ecosystem     (todas las apps, skills, READMEs)
    └── notebook: ElitePass-Reservas      (arquitectura técnica, CLAUDE.md)
    └── notebook: ElitePass-Security      (hardening, geoblock, CrowdSec)
    └── MCP: notebooklm-mcp (roomi-fields) en servidor

Memoria Claude                        ← contexto sesión a sesión
    └── MEMORY.md (índice)
    └── feedback_*.md, project_*.md
```

## Mapa final de aplicaciones

| Carpeta | PM2 | Puerto | GitHub | Versión |
|---|---|---|---|---|
| elitepass-reservas | elitepass-reservas | 3000 | danny9001/sis_res | 1.4.76 |
| elitepass-pos | elitepass-pos | — | danny9001/sis-pos-erp | 1.1.6 |
| elitepass-payments | elitepass-payments | 3100 | danny9001/elitepass-payments | 1.0.2 |
| elitepass-noti-telegram | elitepass-noti-telegram | 3200 | danny9001/elitepass-noti-telegram | 1.0.3 |
| elitepass-monitor | elitepass-monitor | — | danny9001/server-monitor-bot | 1.0.3 |

## Reglas permanentes establecidas

### Nomenclatura
- Patrón: `elitepass-<nombre>` en carpeta, PM2 y `name` de `package.json`
- Cualquier app nueva debe seguir este patrón desde el inicio

### Versionado (cada push a GitHub)
- PATCH +1 por cada push
- Al llegar PATCH a 99 → MINOR +1, PATCH = 0
- Flujo: bump package.json → commit → push

### Seguridad git (verificar antes de cada push)
```bash
git diff --cached --name-only | grep -E "\.env|authorized-users|\.pem|\.key"
```
Archivos prohibidos: `.env.*`, `authorized-users.json`, `*.pem`, `*.key`, `*.crt`

### Sincronización de skills
- Editar SOLO en: `elitepass-monitor/openspec/skills/`
- Push a: `danny9001/skill-elite-pass-knowledge`
- Pull en otros proyectos: `cd openspec/skills && git pull origin main`
