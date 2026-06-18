# Tasks — Estandarización ElitePass 2026-06-05

## Nomenclatura
- [x] Renombrar carpeta `club-administrator` → `elitepass-reservas`
- [x] Renombrar carpeta `telegram-notification-service` → `elitepass-noti-telegram`
- [x] Renombrar carpeta `server-monitor` → `elitepass-monitor`
- [x] Actualizar PM2: eliminar entradas viejas y registrar con nuevos nombres
- [x] Actualizar `ecosystem.config.js` en `elitepass-noti-telegram` (cwd + nombre)
- [x] Crear `ecosystem.config.js` en `elitepass-monitor`
- [x] Actualizar `package.json` name en `elitepass-monitor` y `elitepass-noti-telegram`
- [x] Actualizar referencias en `monitor-service.ts` (rutas y nombre PM2)
- [x] Actualizar nginx `/var/www/club-administrator` → `/var/www/elitepass-reservas`
- [x] Actualizar git remote `origin` en `elitepass-reservas` (apuntaba a carpeta vieja)
- [x] `pm2 save` para persistir configuración

## Seguridad git
- [x] Eliminar `authorized-users.json` del tracking en `elitepass-monitor`
- [x] Eliminar `frontend/.env.production` del tracking en `elitepass-payments`
- [x] Eliminar `frontend/.env.production` del tracking en `elitepass-pos`
- [x] Crear `.gitignore` completo en `elitepass-payments` (no tenía ninguno)
- [x] Reforzar `.gitignore` en `elitepass-reservas` (`.env.*` faltaba)
- [x] Reforzar `.gitignore` en `elitepass-pos` (cubrir todos los patrones `.env.*`)
- [x] Ampliar `.gitignore` en `elitepass-noti-telegram`
- [x] Push de correcciones de seguridad a todos los repos

## Versionado
- [x] Establecer versiones base en todos los proyectos según historial de commits
- [x] Documentar regla de versioning en memoria Claude y skills
- [x] Push de versiones actualizadas a todos los repos

## Skills y conocimiento
- [x] Sincronizar skills de `elitepass-reservas` con los de `elitepass-monitor`
- [x] Agregar sección REGLAS PERMANENTES en `skill-elitepass-organizer.md`
- [x] Agregar sección DEPLOY OBLIGATORIO y SEGURIDAD GIT en `skill-elitepass-server-monitor.md`
- [x] Inicializar `elitepass-monitor/openspec/skills/` como repo git → `skill-elite-pass-knowledge`
- [x] Inicializar `elitepass-reservas/openspec/skills/` como repo git → `skill-elite-pass-knowledge`
- [x] Push de todos los cambios al repo central de skills
- [x] Documentar en memoria Claude: regla de sincronización de skills

## NotebookLM
- [x] Evaluar opciones de MCP para NotebookLM
- [x] Instalar `notebooklm-mcp` v2.0.1 en servidor (@roomi-fields/notebooklm-mcp)
- [x] Instalar Chromium headless via patchright (113.2 MiB en ~/.cache/ms-playwright/)
- [x] Autenticar con cuenta Google Premium (state.json copiado desde Windows, 23 cookies válidas)
- [x] Configurar Claude MCP en `~/.claude.json` (scope: user, global)
- [ ] Crear notebooks en NotebookLM por proyecto
- [ ] Cargar fuentes: READMEs, skills, CLAUDE.md, specs
- [ ] Documentar flujo de sincronización automática

## OpenSpec
- [x] Actualizar change `telegram-notification-service` con nombre nuevo
- [x] Documentar este change (standardization-2026-06-05)
- [x] Actualizar specs con nombres nuevos (`club-administrator` → `elitepass-reservas`)
