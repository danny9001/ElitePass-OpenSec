# Proposal: Nueva Plataforma elitepass-monitor2 (ChatOps & Remediation Engine)

## Business Intent
El objetivo es diseñar y planificar el desarrollo de **elitepass-monitor2**, un nuevo sistema de monitoreo especializado que reemplazará al actual bot básico de monitoreo. En esta nueva arquitectura, **Zabbix** absorbe la responsabilidad del motor de recopilación de telemetría de infraestructura, historización y disparo de alarmas. Por su parte, **elitepass-monitor2** se rediseña desde cero como un **Orquestador de ChatOps, Ejecutor de Runbooks de Autocuración (Self-Healing) y Exportador de Métricas de Negocio**, actuando como el cerebro operativo interactivo del ecosistema ElitePass.

## Scope
1. **Absorción en Zabbix:** Migrar las reglas básicas de hardware (CPU, RAM, Disco) y de servicios (PostgreSQL, Docker) de `elitepass-monitor` hacia plantillas nativas de Zabbix Agent 2 y Web Scenarios.
2. **Orquestador de ChatOps (elitepass-monitor2):** Un microservicio NodeJS/TypeScript dedicado que implementa un bot interactivo bidireccional y un servidor HTTP para control del servidor y despliegue (/deploy, /logs, /firewall).
3. **Runbook Remediation Engine (Self-Healing):** Un motor de reglas especializado en resolver incidentes reportados por Zabbix mediante la ejecución de scripts autorizados (ej: reiniciar clusters de PM2, purgar colas Redis, matar bloqueos en Postgres).
4. **App Metrics Exporter (SSO & Licencias):** Exponer un endpoint seguro `/api/v1/metrics` para que Zabbix Server consuma métricas de negocio (ej. usuarios activos en identity, licencias por expirar, colas de notificaciones Telegram).
5. **Auditoría e Integridad:** Registrar todas las acciones automáticas y comandos manuales en el Audit Log centralizado de `elitepass-identity`.
