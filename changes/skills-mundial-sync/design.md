# Design: Sincronización skills + openspec VM02

## Arquitectura de conocimiento resultante

```
GitHub (skill-elite-pass-knowledge)   ← fuente de verdad compartida
    VM00: /home/soporte/elitepass-monitor/openspec/skills/  ← EDITAR aquí
    VM02: /home/soporte/skill-elite-pass-knowledge/         ← solo pull

Sync VM02: cron 4:05 AM → /home/soporte/sync-skills.sh → git pull

OpenSpec VM00: /home/soporte/openspec/specs/ + changes/
OpenSpec VM02: /home/soporte/openspec/specs/ + changes/
```

## Decisiones

### Skill skill-elitepass-mundial (nuevo)
Documenta stack específico de VM02: Next.js 16, pg directo, WebAuthn, web-push,
arquitectura dual Podman+PM2, deploy con podman build, DB en contenedor.

### Arquitectura real VM02 (corregida)
- Podman: nginx-lb(3001)→app_1/app_2→postgres — PRIMARIO (tráfico real)
- PM2: apuestas-mundial en 3002 — instancia paralela independiente
- Ambos corriendo simultáneamente

### ChunkLoadError POS
Causa: service worker cachea chunks del build anterior; nuevo index.html pide chunks con hash nuevo.
Fix: ChunkErrorBoundary en main.tsx que detecta el error y llama window.location.reload() una vez.
