# Tasks: Nueva Plataforma elitepass-monitor2 (ChatOps & Remediation Engine)

- [ ] **Fase 1: Migración e Integración de Infraestructura en Zabbix**
  - [ ] Habilitar y validar el monitoreo nativo en Zabbix Server usando Zabbix Agent 2 (CPU, RAM, Disco, Docker, Postgres).
  - [ ] Configurar alertas de infraestructura en Zabbix Server.
  - [ ] Crear escenarios web sintéticos en Zabbix Server para las URLs del ecosistema.

- [ ] **Fase 2: Estructura base de elitepass-monitor2**
  - [ ] Crear el directorio `/home/soporte/elitepass-monitor2` e inicializar el proyecto TypeScript.
  - [ ] Configurar variables de entorno `.env` unificadas (Zabbix Server IP, Zabbix API credentials, Telegram token, Identity DB connection).
  - [ ] Configurar el ORM Prisma o consultas nativas para acceder a la base de datos de Identity y verificar membresías/licencias.

- [ ] **Fase 3: Receptor HTTP, ChatOps y Zabbix API Client**
  - [ ] Programar el servidor HTTP (ej. Express o http nativo) escuchando en el puerto `3500` expuesto en `/api/v1/zabbix-alert`.
  - [ ] Codificar el cliente JSON-RPC para interactuar con la API de Zabbix Server (`event.acknowledge`, `trigger.get`).
  - [ ] Implementar la interfaz de Telegram con botones interactivos para recibir alertas de Zabbix, y responder con reconocimiento en la API.
  - [ ] Migrar y refactorizar comandos de control esenciales (`/status`, `/npm`, `/deploy`, `/logs`, `/firewall`).

- [ ] **Fase 4: Motor de Runbooks y Autocuración (Self-Healing)**
  - [ ] Programar el núcleo del motor de runbooks con soporte para reintentos y cooldowns.
  - [ ] Escribir los runbooks especializados:
    - [ ] `pm2-app-restart`: Reinicio seguro de workers PM2 ante alertas de caída.
    - [ ] `pg-connection-killer`: Liberación de conexiones inactivas/colgadas en Postgres.
    - [ ] `disk-space-cleaner`: Purgado inteligente de logs y caches temporales si el disco supera el 85%.

- [ ] **Fase 5: Exportador de Métricas de Negocio**
  - [ ] Implementar la ruta `/api/v1/metrics` con seguridad por API Key.
  - [ ] Programar consultas a la base de datos de Identity para reportar sesiones activas, licencias por vencer y colas de pagos.
  - [ ] Configurar el item HTTP Agent en Zabbix Server para consumir este endpoint periódicamente.

- [ ] **Fase 6: Puesta en Marcha y Desmantelamiento**
  - [ ] Desplegar `elitepass-monitor2` bajo PM2 (name: `elitepass-monitor-v2`, id: 8).
  - [ ] Integrar el registro de auditoría automática (`SSO LogAudit`) en cada comando y acción correctiva ejecutada por el bot.
  - [ ] Apagar y desinstalar el antiguo proceso PM2 `elitepass-monitor` (id: 1) una vez validada la estabilidad de la nueva plataforma.
