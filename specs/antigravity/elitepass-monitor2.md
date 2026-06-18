# Specification: elitepass-monitor2 (ChatOps & Remediation Engine)

Orquestador operativo y de autocuración del ecosistema **Antigravity**. Actúa como la interfaz interactiva de control (ChatOps) y el ejecutor automático de runbooks ante alarmas despachadas por **Zabbix Server**, además de servir como exportador de métricas de negocio para la analítica del ecosistema.

---

## 1. Responsabilidades Compartidas (Matriz de Control)

| Responsabilidad | Sistema | Descripción |
|---|---|---|
| **Telemetría de Hardware** | Zabbix Agent 2 | CPU, RAM, almacenamiento, red en VM00 y VM02. |
| **Monitoreo de Base de Datos** | Zabbix Agent 2 | Métricas nativas de PostgreSQL, PgBouncer y transacciones. |
| **Salud de Contenedores** | Zabbix Agent 2 | Estado y recursos de docker daemon y contenedores activos. |
| **Escenarios Web (HTTP)** | Zabbix Server | Disponibilidad externa, latencia y expiración de certificados SSL. |
| **Alertas e Historial** | Zabbix Server | Generación de triggers, almacenamiento histórico, gráficas y SLAs. |
| **ChatOps (Comandos)** | elitepass-monitor2 | Interfaz Telegram interactiva para `/status`, `/deploy`, `/logs`, `/firewall`. |
| **Autocuración (Self-Healing)**| elitepass-monitor2 | Ejecución de runbooks locales automáticos al recibir webhooks de Zabbix. |
| **Métricas de Aplicación** | elitepass-monitor2 | Exponer `/api/v1/metrics` para Zabbix con datos de base de datos de negocio. |
| **Logs & SSH Watcher** | elitepass-monitor2 | Auditoría de accesos auth.log y reenvío de trappers de seguridad a Zabbix. |

---

## 2. Definición de Escenarios de Autocuración (Self-Healing)

### Escenario 1: Caída de Worker PM2
- **Given** Zabbix Server detecta que una aplicación PM2 (ej: `elitepass-pos`) está inactiva y genera un trigger de severidad `Alta`.
- **When** Zabbix envía la alerta vía Webhook a `elitepass-monitor2`.
- **Then** el orquestador valida el cooldown y ejecuta localmente `pm2 restart elitepass-pos`.
- **And** envía un mensaje HTML a Telegram detallando la incidencia detectada y el resultado de la recuperación automática.

### Escenario 2: Agotamiento de Conexiones en Postgres/PgBouncer
- **Given** Zabbix detecta que el pool de conexiones de PgBouncer está cerca del límite (>90%) y emite una alerta.
- **When** `elitepass-monitor2` recibe el webhook del evento.
- **Then** el orquestador ejecuta una consulta de mantenimiento para terminar conexiones inactivas en estado `idle in transaction` con más de 10 minutos de antigüedad.
- **And** notifica en el canal de soporte el número de conexiones liberadas.

### Escenario 3: Alerta de Almacenamiento Alto (>85%)
- **Given** Zabbix Agent reporta un uso de disco mayor al 85% en `/`.
- **When** `elitepass-monitor2` recibe la alerta de capacidad de Zabbix.
- **Then** el orquestador inicia el runbook `disk-space-cleaner` para purgar logs antiguos comprimidos (`.gz`) en `/var/log` y archivos temporales de PM2, liberando espacio de forma segura.

---

## 3. Especificación de Métricas de Negocio (`/api/v1/metrics`)

El endpoint expuesto en `elitepass-monitor2` retornará métricas de aplicación que no pueden ser obtenidas directamente por Zabbix Agent a nivel de sistema operativo:

```json
{
  "sessions": {
    "sso_active_users_12h": "integer",
    "passkeys_total": "integer"
  },
  "licenses": {
    "active_companies_licenses": "integer",
    "expiring_licenses_15d": "integer"
  },
  "payments": {
    "pending_webhooks_sync": "integer",
    "gateway_health_status": "string (UP|DOWN)"
  }
}
```

---

## 4. Auditoría y Trazabilidad (Audit Logs)

Toda acción correctiva (autocuración) y comando enviado manualmente por el administrador a través de Telegram (`/deploy`, `/npm restart`, `/fw`) debe ser registrado en la base de datos de auditoría de Identity:
- **Campos del AuditLog:**
  - `evento`: `"monitor_runbook_executed"` | `"monitor_admin_command"`
  - `userId`: ID del administrador de Telegram que emitió el comando (o `"system_self_healing"` para acciones automáticas).
  - `metadata`: Objeto JSON con el comando ejecutado, trigger de Zabbix asociado, estado final y salida del shell.
