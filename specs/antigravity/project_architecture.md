---
name: project-architecture
description: "ElitePass Mundial corre en PM2 bare-metal cluster (4 procesos). Arquitectura actual: puerto 3002 PM2."
metadata: 
  node_type: memory
  type: project
  originSessionId: 2d589995-741a-43f1-9c51-d3d0676f424e
---

## Stack Actual (ACTUALIZADO 2026-06-12)

El proyecto **ElitePass Mundial** corre en **PM2bare-metal** con arquitectura **cluster mode** (4 procesos).

### PM2 Cluster (PRODUCCIÓN — tráfico real)

- **PM2 ids 9, 11, 12, 13** — `elitepass-mundial` 
- **Modo:** cluster (4 procesos)
- **Puerto:** 3002
- **Node.js:** v16.2.6
- **Uptime:** ~14 minutos (activo)
- **Estado:** online ✅
- Usa `.next/` (build Next.js standalone) + `node_modules/` en disco
- **Comando start:** `npm start -- -p 3002` o equivalente
- **nginx bare-metal** (443) → proxy a `127.0.0.1:3002`

### Archivo .env

- `.env.local` — variables para PM2 bare-metal en `/home/soporte/elitepass-mundial/`
- Variables DB: `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`
- **PostgreSQL:** connexión local (probablemente en localhost)

---

## Deploy

```bash
# PM2 bare-metal (en producción)
cd /home/soporte/elitepass-mundial
git pull origin main
npm run build  # Genera .next/ standalone
pm2 restart elitepass-mundial

# O restart de todos los procesos
pm2 restart 9 11 12 13
```

## Archivos Importantes

- `.env.local` — vars de entorno para PM2
- `package.json` — `npm start` lanza en cluster mode (o configurado en PM2)
- `.next/` — build standalone en disco
- `node_modules/` — dependencias en disco

## Notas Críticas

- **PM2 en cluster mode** = 4 procesos loadbalanceados automáticamente
- **.next/ SÍ existe** en disco — build already generated
- **node_modules/ SÍ existe** en disco
- **DB pool:** postgres local con variables individuales (no DATABASE_URL)
- **No Podman en producción** — es solo PM2 bare-metal

---

## Verificación de Estado

```bash
pm2 list                    # Ver status de procesos
pm2 logs elitepass-mundial  # Ver logs en tiempo real
curl http://localhost:3002  # Verificar app responde
```

**Why:** PM2 cluster mode proporciona alta disponibilidad (4 procesos) y restart automático en case de crash.

**How to apply:** Para cambios de código: `npm run build && pm2 restart elitepass-mundial` (o IDs específicos).
