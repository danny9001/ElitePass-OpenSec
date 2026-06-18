# Proposal: Synchronize Remote Mundial Server (VM02) and Inspect All Systems

## Business Intent
We need to verify that all systems across our ecosystem are running securely and optimally. This involves:
1. Inspecting all local systems (`elitepass-reservas`, `elitepass-pos`, `elitepass-identity`, etc. on VM00) and generating a checklist verifying security, PM2 status, Nginx configurations, database migrations, and SSL certificate health.
2. Connecting to the remote server `10.0.0.7` (VM02-MUNDIAL) via SSH on port `5001` using `sshpass`.
3. Comparing and synchronizing the `openspec` memories, specs, and changes between VM00 and VM02.
4. Pulling or pushing updates to `danny9001/skill-elite-pass-knowledge` on both servers so their skills directory remains perfectly aligned.
5. Performing a total system inspection (Docker/Podman, PM2, Disk usage, logs, CrowdSec, etc.) and outlining any discrepancies or failures.

## Target Scope
- Global specs folder: `/home/soporte/openspec` (both VM00 and VM02).
- Skills repository clones: `/home/soporte/elitepass-reservas/openspec/skills` and `/home/soporte/skill-elite-pass-knowledge` (on VM02).
- System services check on both hosts.
