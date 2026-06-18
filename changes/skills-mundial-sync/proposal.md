# Proposal: Sincronización de skills y openspec con VM02-MUNDIAL

## Business Intent
VM02-MUNDIAL no tenía openspec ni sus skills estaban actualizados con la info del ecosistema completo.
Los agentes AI en VM02 operaban con memoria desactualizada (decía Podman sin PM2, ahora sabemos que ambos corren).
Necesitábamos que ambos servidores tengan la misma fuente de verdad de reglas y arquitectura.

## Scope
1. Actualizar skills con información de VM02-MUNDIAL (servidores, nomenclatura, deploy, arquitectura dual Podman+PM2)
2. Crear skill-elitepass-mundial.md con todo el stack específico de VM02
3. Crear estructura openspec en VM02 (/home/soporte/openspec/specs/ + changes/)
4. Actualizar MEMORY.md de VM02 con arquitectura real (Podman primario en 3001, PM2 en 3002)
5. Instalar cron en VM02 y crear sync-skills.sh para pull diario desde skill-elite-pass-knowledge
6. Fix ChunkLoadError en elitepass-pos: error boundary que auto-recarga al detectar SW stale
