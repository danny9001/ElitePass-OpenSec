# Estandarización del Ecosistema ElitePass — 2026-06-05

## Business Intent

Unificar la nomenclatura, versionado, seguridad git y sincronización de conocimiento
de todas las aplicaciones del ecosistema ElitePass para reducir fricción operacional
y errores causados por inconsistencias históricas.

## Errores encontrados (root cause analysis)

### 1. Nomenclatura inconsistente
- La app principal se llamaba `club-administrator` (carpeta, PM2, package.json)
  mientras que en algunas partes ya era `elitepass-reservas`.
- El servicio de notificaciones era `telegram-notification-service` sin prefijo ElitePass.
- El monitor era `server-monitor` sin prefijo.
- **Impacto:** referencias cruzadas rotas en monitor-service.ts, skills y openspec.

### 2. Archivos sensibles en git
- `authorized-users.json` (IDs de admins Telegram) trackeado en `elitepass-monitor`.
- `frontend/.env.production` con URLs internas trackeado en `elitepass-payments` y `elitepass-pos`.
- `elitepass-payments` no tenía `.gitignore` — cualquier `git add .` hubiera subido `.env` reales.
- **Impacto:** exposición de datos de administradores y topología interna.

### 3. Skills desincronizadas
- Cada proyecto tenía su propia copia de los skills con versiones diferentes.
- El repo central `skill-elite-pass-knowledge` existía pero no se usaba como fuente de verdad.
- `elitepass-reservas/openspec/skills/` todavía tenía referencias a `club-administrator`.
- **Impacto:** Claude y Gemini/Antigravity operaban con reglas distintas entre proyectos.

### 4. Versionado manual y sin regla
- Ningún proyecto tenía una regla de versioning por push.
- Las versiones en `package.json` estaban desactualizadas respecto al historial real de commits.
- **Impacto:** imposible saber qué versión corre en producción mirando el package.json.

### 5. OpenSpec desactualizado
- Changes de `telegram-notification-service` y `specs/` con nombres viejos (`club-administrator`).
- No había documentación de reglas permanentes del ecosistema en ningún formato operacional.

## Scope de la solución

1. Renombrar todas las carpetas, procesos PM2 y package.json al patrón `elitepass-<nombre>`.
2. Eliminar archivos sensibles del tracking git y reforzar `.gitignore` en todos los proyectos.
3. Convertir `skill-elite-pass-knowledge` en fuente de verdad única para skills.
4. Establecer regla de versionado: cada push = PATCH+1, al 99 → MINOR+1.
5. Documentar reglas permanentes en skills y memoria de agentes (Claude + Antigravity).
6. Integrar NotebookLM via MCP para análisis centralizado sin ocupar espacio en servidor.
