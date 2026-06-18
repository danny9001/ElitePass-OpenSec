# Design: Synchronize Remote Mundial Server (VM02) and Inspect All Systems

## Execution Strategy

### 1. Local System Inspection (VM00)
We will execute diagnostics commands to verify:
- **PM2 Processes**: Check status of `elitepass-reservas`, `elitepass-pos`, `elitepass-identity`, `elitepass-payments`, `elitepass-noti-telegram`.
- **Nginx configuration**: Check rate limit zones, HSTS, gzip, and Brotli module status.
- **Security services**: Verify `crowdsec`, `crowdsec-firewall-bouncer`, and `geoblock-multiport`.
- **Database migrations**: Check status of database schema migrations.
- **SSL Certificates**: Verify status and expiry of the certificates for `reservas.genial-it.net`, `id.genial-it.net`, etc.

### 2. Remote Server Connection (VM02)
We will execute remote commands over SSH using `sshpass`:
- SSH credentials: `sshpass -p 'Password2012' ssh -o StrictHostKeyChecking=no -p 5001 soporte@10.0.0.7`
- Diagnostic checks on VM02:
  - PM2 apps: `apuestas-mundial`
  - PostgreSQL status
  - Disk space, memory, and CPU utilization
  - Nginx configurations

### 3. OpenSpec & Skills Synchronization
- **VM00 to VM02 OpenSpec sync**:
  - We will check if `/home/soporte/openspec/` exists on VM02 and what files are in there.
  - If there are new specs or changes, we can copy them over SSH or log them in a status diff.
- **Skills synchronization**:
  - Check the remote clone of `skill-elite-pass-knowledge` at `/home/soporte/skill-elite-pass-knowledge`.
  - Pull the latest commits that we pushed in the previous turn.
  - Sychronize the local skills repository clones of both VM00 and VM02.
  - If any modifications are found on VM02, resolve conflicts, integrate them into the global skills, and push/pull them.
