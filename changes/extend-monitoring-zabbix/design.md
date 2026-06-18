# Design: Extender el Sistema de Monitoreo con Zabbix

## Modelo de Coexistencia de Monitoreo

El sistema de monitoreo actual (`elitepass-monitor` / `monitor_servicios_nodejs.sh`) es directo y muy rápido para alertas a Telegram, pero carece de persistencia de métricas y gráficos. Zabbix se integrará como una capa superior sin quitar ni alterar el script actual:

```
[ Infraestructura VM00 ]
   │
   ├──► elitepass-monitor (Cron) ──► Telegram Bot (Alertas instantáneas)
   │
   └──► Zabbix Agent 2 (Daemon)
           │
           ▼ (Métricas: CPU, RAM, Docker, PostgreSQL, PM2)
       [ Zabbix Server ] ──► Dashboards / Histórico / Alertas Complejas (Grafana / SLA)
```

---

## Decisiones Técnicas y Arquitectura

### 1. Zabbix Agent 2 en VM00
Se instalará `zabbix-agent2` porque incluye soporte nativo compilado en Go para plugins como PostgreSQL, Docker y HTTP, requiriendo menos scripts personalizados en el cliente.
- **Docker Monitoring:** Zabbix Agent se añadirá al grupo `docker` local para poder consultar el socket `/var/run/docker.sock` de forma segura.
- **PM2 Metrics via UserParameter:**
  Para monitorear los procesos de Node.js administrados por PM2, agregamos una directiva en `/etc/zabbix/zabbix_agent2.d/pm2.conf`:
  ```ini
  UserParameter=pm2.apps.count,pm2 jlist | jq '. | length'
  UserParameter=pm2.apps.offline,pm2 jlist | jq '[.[] | select(.pm2_env.status != "online")] | length'
  ```
  Esto permite a Zabbix alarmar si el número de aplicaciones en línea disminuye o si alguna app se cae.

### 2. Monitoreo de PostgreSQL y PgBouncer
Zabbix utilizará el template oficial `PostgreSQL by Zabbix Agent 2`.
- Se creará un usuario de solo lectura en la base de datos para Zabbix:
  ```sql
  CREATE USER zbx_monitor WITH PASSWORD 'tu_password_segura';
  GRANT pg_monitor TO zbx_monitor;
  ```
- El agente se comunicará con PostgreSQL local en el puerto `5432` y con PgBouncer en el puerto `6432` para extraer métricas de pools y transacciones por segundo.

### 3. Escenarios Web (Synthetic Monitoring) y Certificados SSL
Configurado directamente en Zabbix Server (sin agente en VM00) para comprobar externamente la salud de las aplicaciones:
- **Verificación de URL:** HTTP GET a las URLs principales, validando código de estado `200` y tiempo de respuesta inferior a 1.5s.
- **Certificados SSL:** Trigger configurado para alertar cuando un certificado SSL (Nginx/Let's Encrypt) esté a menos de 10 días de expirar.

### 4. Flujo de Alertas Telegram desde Zabbix
Zabbix Server utilizará un canal de comunicación (Media Type) nativo de Telegram.
- Se configurará para usar el mismo bot token de `elitepass-monitor` y el mismo Chat ID para unificar los canales de notificaciones del equipo soporte.
