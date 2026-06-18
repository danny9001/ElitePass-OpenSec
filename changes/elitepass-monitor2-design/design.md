# Design: Nueva Plataforma elitepass-monitor2 (ChatOps & Remediation Engine)

## Arquitectura de Integración (Zabbix + elitepass-monitor2)

El diseño propuesto separa el **Motor de Datos (Zabbix)** del **Motor de Acción y Control (elitepass-monitor2)**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                              ECOSISTEMA                                │
│   [ VM00 Host OS ] ──► Zabbix Agent 2  ──► Envia Métricas              │
│   [ PostgreSQL ]                                                       │
│   [ PM2 Apps ]     ◄── elitepass-monitor2 ◄── Comando / Runbook        │
└────────────────────────────────────────────────────────────────────────┘
                              │         ▲
                              ▼         │ Webhook Alerta / API Control
                      ┌──────────────┐  │
                      │Zabbix Server │ ─┘
                      └──────────────┘
```

---

## Componentes y Módulos de elitepass-monitor2

### 1. Módulo 1: Receptor de Alertas y ChatOps (Telegram Bot)
- **Bot Bidireccional:** Permite a los administradores autorizados interactuar con el servidor. Soporta comandos interactivos `/status`, `/npm`, `/deploy`, `/logs`, `/firewall`.
- **Integración con Zabbix API:** Cuando llega una alerta de Zabbix al bot, este incluye un botón inline `[Reconocer Alerta]`. Al presionarlo, el bot se autentica en Zabbix Server vía JSON-RPC y ejecuta `event.acknowledge` para silenciar la alarma.

### 2. Módulo 2: Motor de Autocuración y Runbooks (Self-Healing Engine)
El orquestador de runbooks traduce las alertas de Zabbix en acciones correctivas locales en VM00 y VM02 de forma segura:
- **Estructura de un Runbook:**
  ```typescript
  interface Runbook {
    triggerName: string;      // Identificador de alerta de Zabbix
    maxRetries: number;       // Evitar loops infinitos (ej. 3)
    cooldownMs: number;       // Tiempo de espera entre ejecuciones
    action: () => Promise<void>; // Lógica de remediación
  }
  ```
- **Ejemplos de Runbooks de Autocuración:**
  * **Trigger: `PM2 app offline`** ──► Ejecuta `pm2 restart <app_name>`.
  * **Trigger: `PgBouncer connection pool exhausted`** ──► Ejecuta un script de Postgres para matar queries inactivas o colgadas en `state = 'idle in transaction'`.
  * **Trigger: `Redis memory high`** ──► Limpia colas de notificaciones procesadas o memoria temporal obsoleta.
  * **Trigger: `Disk space high (>85%)`** ──► Puga automáticamente logs antiguos comprimidos (`.gz`) en `/var/log` y `/home/soporte/.pm2/logs`.

### 3. Módulo 3: Exportador de Métricas de Negocio (SSO & Apps)
Un endpoint HTTP expuesto en `elitepass-monitor2` (ej: `/api/v1/metrics`) protegido con API Key. Zabbix Server lo consumirá periódicamente usando un **HTTP Agent Item**.
Retornará estadísticas de negocio del ecosistema consultando la base de datos central:
```json
{
  "identity": {
    "total_users": 1500,
    "active_sessions_12h": 142,
    "passkeys_registered": 89,
    "active_licenses": 45,
    "expired_licenses_warning": 3
  },
  "payments": {
    "transactions_pending_sync": 0,
    "gateway_health": "UP"
  },
  "reservas": {
    "total_bookings_today": 320,
    "pending_ticket_sync": 0
  }
}
```

### 4. Módulo 4: Monitor de Auditoría y Seguridad
- **Auth.log Watcher:** Vigila intentos SSH fallidos e inicios de sesión exitosos. Reporta eventos de seguridad directamente a Zabbix utilizando el protocolo **Zabbix Trapper** en lugar de enviar mensajes de texto planos, permitiendo a Zabbix centralizar y graficar los incidentes de seguridad de todos los servidores.
- **Audit Logging:** Cada acción realizada por el bot (sea autocuración o comando de administrador como `/deploy`) generará un registro de auditoría en la tabla `AuditLog` de `elitepass-identity` para mantener un registro histórico inmutable de la administración del servidor.
