# Design: System Inspection & OpenSpec/Skills Sync (VM00 - VM02)

## 1. Local Diagnostics (VM00)
- **Host Info**: Ubuntu 24.04 ARM64 (`10.0.0.5`).
- **PM2**: Check `elitepass-reservas`, `elitepass-pos`, `elitepass-identity`, `elitepass-payments`, `elitepass-noti-telegram`.
- **System Metrics**: Disk usage (`df -h`), memory usage (`free -m`), CPU load.
- **Nginx & SSL**: Inspect `/etc/nginx/sites-enabled/` and check SSL certificate expiration dates.
- **Security**: Verify `crowdsec`, `crowdsec-firewall-bouncer` statuses, and `ipset` rules.
- **Database**: Check PostgreSQL system status and database sizes.

## 2. Remote Diagnostics (VM02)
- **Host Info**: Ubuntu 24.04 x86_64 (`10.0.0.7`).
- **SSH Connectivity**: Connection over port `5001` with `sshpass -p 'Password2012' ssh -o StrictHostKeyChecking=no -p 5001 soporte@10.0.0.7`.
- **PM2**: Check `apuestas-mundial`, `apuestas-scheduler`, `server-monitor-bot`.
- **System Metrics**: Disk usage (especially root partition `/`), memory, CPU.
- **PostgreSQL**: Check Postgres service health and direct local connection.
- **Nginx**: Inspect `/etc/nginx/sites-enabled/mundial.genial-it.net` configuration.

## 3. OpenSpec & Skills Synchronization Design
- **OpenSpec Sync**:
  - Compare the list of files in `/home/soporte/openspec/` on VM00 and VM02.
  - Since VM00 is the main control node, copy any missing spec or change directories to/from VM02 using `scp` or `rsync` with `sshpass`.
- **Skills Sync**:
  - The repository `danny9001/skill-elite-pass-knowledge` is cloned locally on VM00 in `/home/soporte/elitepass-pos/openspec/skills/` (and potentially other places) and on VM02 in `/home/soporte/skill-elite-pass-knowledge`.
  - We will check git statuses, push any local updates from VM00 to GitHub, and pull them on VM02 to ensure synchronization.
  - If there are direct edits on VM02, pull them back or synchronize.
