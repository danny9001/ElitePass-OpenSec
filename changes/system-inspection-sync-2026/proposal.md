# Proposal: System Inspection & OpenSpec/Skills Sync (VM00 - VM02)

## Business Intent
To ensure operational stability, security, and consistent configuration across all servers in the ElitePass ecosystem. This is achieved by:
1. Conducting a comprehensive local inspection on VM00 (Reservas, POS, Identity, Payments, Telegram Notifications, and Nginx/CrowdSec security).
2. Connecting securely to VM02-MUNDIAL (`10.0.0.7:5001`) via SSH to run system diagnostics, resource checks, and service status reviews.
3. Comparing and synchronizing the `openspec` directory and the `danny9001/skill-elite-pass-knowledge` repo on both servers.
4. Enhancing skills documentation if discrepancies or improvements are found.
5. Providing a complete, structured checklist of findings.

## Target Scope
- **VM00 (Local)**:
  - PM2 apps: `elitepass-reservas`, `elitepass-pos`, `elitepass-identity`, `elitepass-payments`, `elitepass-noti-telegram`, `elitepass-monitor`
  - System health (CPU, Memory, Disk)
  - Nginx configurations and SSL certificate expiries
  - Security configuration (CrowdSec, firewall, geoblock)
- **VM02 (Remote Mundial)**:
  - Connection over private IP `10.0.0.7:5001`
  - PM2 apps: `apuestas-mundial`, `apuestas-scheduler`, `server-monitor-bot`
  - System health (Disk space at `/`, memory, CPU)
  - PostgreSQL health
  - Nginx configuration
- **OpenSpec & Skills Sync**:
  - Comparison of `/home/soporte/openspec` (specs and changes)
  - Comparison of `/home/soporte/elitepass-pos/openspec/skills/` (VM00) and `/home/soporte/skill-elite-pass-knowledge` (VM02)
  - Identification of outdated files, merging improvements, and pulling/syncing.
