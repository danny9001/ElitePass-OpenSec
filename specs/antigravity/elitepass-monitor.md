# elitepass-monitor: Agente de Monitoreo y Hardening del Servidor

Guardián de infraestructura del ecosistema **Antigravity**. Monitorea salud del sistema, controla PM2, gestiona el firewall (CrowdSec + ipset geoblock) y expone administración remota completa via Telegram — sin necesidad de SSH.

---

## 1. Core Técnico y Arquitectura

- **Tecnología:** Node.js + TypeScript (ejecutado con `tsx` — sin build)
- **Librerías:** `node-telegram-bot-api` (bot de control) + `node-cron` (planificador)
- **Ejecución:** Directamente en el host OS — accede a `systemctl`, `pm2`, `iptables`, `ipset`, `crowdsec-cli` sin contenedor
- **PM2:** modo **Fork** — sin heap limit configurado (consumo bajo, ~80 MB típico)
- **ID PM2:** 1

### Capas de Monitoreo

```mermaid
graph TD
    Cron5m[Cron cada 5 min] -->|Health Check| Monitor[Monitor Service]
    Cron14h[Cron 14:00 diario] -->|Reporte completo| Monitor
    AuthLog[/var/log/auth.log — SSH Watcher] -->|Detección intrusos| Monitor

    Admin[Admin Telegram Chat] -->|Comandos| Bot[Telegram Bot API]
    Bot -->|Verifica authorized-users.json| Monitor

    Monitor -->|shell exec| OS[Sistema Operativo VM00]
    OS -->|métricas, estado PM2, IPs baneadas| Monitor
    Monitor -->|Alertas / Respuestas| Bot
    Bot -->|Mensaje| Admin
```

---

## 2. Acceso y Control de Seguridad del Bot

### Autorización Multi-nivel

- **Archivo de permisos:** `authorized-users.json` — lista de Telegram chat IDs autorizados (nunca commitear al repo)
- **Chat ID del Admin Maestro:** `379973030` (configurado en `TELEGRAM_CHAT_ID`)
- **Flujo de nuevo usuario:** Si un chat ID no autorizado envía un comando → el bot bloquea la acción y notifica al admin con botones interactivos [Autorizar / Rechazar]

### Stack de Seguridad que el Monitor Gestiona

| Capa | Tecnología | Descripción |
|---|---|---|
| **IDS/IPS** | CrowdSec 1.7.8 | Escenarios: SSH brute-force, web exploits, scanners, credential stuffing |
| **Geoblock Multiport** | ipset + iptables | Filtra puertos 80, 443, 5001. Solo permite Bolivia, Cloudflare, Azure, AWS y Google |
| **SSH** | Puerto 5001 (no 22) | Reducción de superficie de ataque — el monitor lo puede abrir/cerrar temporalmente |
| **Watchdog** | auth.log watcher | Detección en tiempo real de intentos de autenticación SSH fallidos |

---

## 3. Comandos de Operación Soportados

### Recursos y Estado del Sistema

| Comando | Acción |
|---|---|
| `/status` | Resumen completo: CPU, RAM, disco, uptime, procesos PM2 |
| `/cpu` | Uso de CPU por núcleo en tiempo real |
| `/ram` | Uso de memoria RAM y swap |
| `/disco` | Uso de disco por partición |
| `/servicios` | Estado de servicios systemd (nginx, postgresql, redis, pgbouncer) |
| `/procesos` | Top 10 procesos por consumo de CPU/RAM |

### Control de Firewall y Seguridad

| Comando | Acción |
|---|---|
| `/crowdsec` | Lista IPs actualmente baneadas por CrowdSec |
| `/ban <IP>` | Banea una IP inmediatamente vía CrowdSec |
| `/unban <IP>` | Desbanea una IP de CrowdSec |
| `/desbanear <IP>` | Elimina IP del ipset de geoblock (permite entrada temporal) |
| `/abrir_ssh` | Habilita regla temporal de acceso SSH en iptables |
| `/cerrar_ssh` | Cierra acceso SSH vía iptables |

### Control PM2

| Comando | Acción |
|---|---|
| `/npm status` | Lista todos los procesos PM2 con estado, CPU y RAM |
| `/npm list` | Detalle de cada worker PM2 |
| `/npm restart <app>` | Reinicia un proceso PM2 específico |
| `/deploy` | Ejecuta: `git pull` + build + `pm2 restart` del servicio indicado |

---

## 4. Alertas Automáticas

El monitor envía alertas proactivas sin que el admin las solicite:

| Condición | Umbral | Acción |
|---|---|---|
| RAM alta | > 85% | Alerta Telegram con detalle de procesos |
| CPU sostenida | > 90% por 2+ ciclos | Alerta Telegram |
| Proceso PM2 caído | pm2 status = errored/stopped | Alerta inmediata |
| Login SSH exitoso | Cualquier conexión en auth.log | Notificación con IP de origen |
| Intento SSH fallido repetido | 5+ en 60s | Alerta de ataque de fuerza bruta |
| Disco alto | > 80% | Alerta de capacidad |

---

## 5. Despliegue e Infraestructura

- **Puerto:** Ninguno — opera completamente por polling a la API de Telegram (no escucha conexiones entrantes)
- **Proceso PM2:** `elitepass-monitor` — modo **Fork** — reinicio automático (`restart_delay: 5000`)
- **ID PM2:** 1
- **Directorio:** `/home/soporte/elitepass-monitor/`
- **OpenSpec local:** `/home/soporte/elitepass-monitor/openspec/skills/` — source de verdad de skills del ecosistema

### Deploy

```bash
cd /home/soporte/elitepass-monitor
# No requiere build — tsx corre directamente
pm2 restart elitepass-monitor
```

### Archivos Sensibles (NUNCA en git)

- `authorized-users.json` — IDs de Telegram autorizados
- `.env` — token del bot y configuración

### Variables de Entorno

```env
TELEGRAM_BOT_TOKEN="token_del_bot_de_seguridad_telegram"
TELEGRAM_CHAT_ID="379973030"
SERVER_NAME="VM00-Reservas-v2"
HEALTH_INTERVAL="*/5 * * * *"
DAILY_REPORT_TIME="0 14 * * *"
```
