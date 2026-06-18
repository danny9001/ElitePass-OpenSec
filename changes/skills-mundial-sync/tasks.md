# Tasks: Sincronización skills + openspec VM02

## Skills (VM00 → GitHub)
- [x] skill-elitepass-organizer: tabla de servidores VM00+VM02, nomenclatura completa, sync bidireccional
- [x] skill-elite-pass-knowledge: descripción ecosistema completo, checklist VM02, sección mundial en historial
- [x] skill-elitepass-mundial (NUEVO): stack, runtime dual, deploy Podman, DB, nginx, auth, openspec
- [x] Corregir skill-elitepass-mundial: arquitectura dual Podman primario + PM2 3002
- [x] Push a danny9001/skill-elite-pass-knowledge

## VM02 setup
- [x] git pull skill-elite-pass-knowledge (skills actualizados)
- [x] Crear /home/soporte/openspec/specs/apuestas-mundial/spec.md (BDD)
- [x] Crear /home/soporte/openspec/changes/ (estructura vacía lista)
- [x] Instalar cron (no estaba instalado en VM02)
- [x] Crear /home/soporte/sync-skills.sh + cron 4:05 AM
- [x] Actualizar MEMORY.md VM02 con arquitectura real y openspec info
- [x] Actualizar project_architecture.md VM02 con runtime dual correcto

## Fix POS
- [x] ChunkErrorBoundary en main.tsx — auto-reload una vez al detectar ChunkLoadError
- [x] Rebuild dist + bump 1.1.8 → 1.1.9 + push
