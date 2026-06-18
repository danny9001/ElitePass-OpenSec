# Design: Sistema Híbrido de Monitoreo y ChatOps (Zabbix + ElitePass Monitor)

## Arquitectura del Sistema Híbrido

El sistema unificado operará bajo una arquitectura de lazo cerrado (Closed Loop) combinando el motor de triggers de Zabbix y las herramientas de ejecución local del bot:

```
┌─────────────────┐             Trigger              ┌────────────────────────┐
│  Zabbix Server  │ ───────────────────────────────► │ elitepass-monitor Bot  │
└─────────────────┘                                  └────────────────────────┘
         ▲                                                       │
         │ API: Acknowledge/Resolve                              │ Envía mensaje HTML
         │                                                       ▼
┌─────────────────┐       Ejecuta acción local       ┌────────────────────────┐
│  Zabbix Agent2  │ ◄─────────────────────────────── │  Telegram (Chat / App) │
└─────────────────┘     (ej. pm2 restart app)        └────────────────────────┘
                                                         (Admin presiona botón)
```

---

## Flujos de Trabajo e Integraciones

### 1. Endpoint de Webhook para Zabbix (`/api/v1/zabbix-alert`)
Expondremos un servidor HTTP básico en `elitepass-monitor` (usando `http` o `express` si está disponible) configurado en un puerto exclusivo (ej: `3250`).
- **Seguridad:** El endpoint validará un token secreto en la cabecera `Authorization: Bearer <secret_token>`.
- **Payload esperado de Zabbix:**
  ```json
  {
    "event_id": "{EVENT.ID}",
    "trigger_name": "{TRIGGER.NAME}",
    "trigger_severity": "{TRIGGER.SEVERITY}",
    "host": "{HOST.NAME}",
    "item_value": "{ITEM.LASTVALUE}",
    "app_name": "{ITEM.KEY}" 
  }
  ```
- **Acción:** El bot formatea la alerta con colores según severidad y añade botones en línea (*Inline Keyboard*):
  - `[✅ Reconocer Alerta]` (llama a la API de Zabbix para marcar como reconocido).
  - `[🔄 Reiniciar App]` (si la alerta es por PM2 o Docker caído).
  - `[📄 Ver Últimos Logs]` (retorna los últimos 20 logs).

### 2. Integración con la API de Zabbix (JSON-RPC)
Para interactuar con Zabbix Server, el bot implementará llamadas al endpoint `https://zabbix-server.internal/zabbix/api_jsonrpc.php` utilizando fetch:
- **Autenticación:**
  ```json
  {
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": { "username": "admin", "password": "zabbix_password" },
    "id": 1
  }
  ```
- **Comando `/alertas` (Telegram -> Zabbix):**
  Consulta triggers con estado de problema activo:
  ```json
  {
    "jsonrpc": "2.0",
    "method": "trigger.get",
    "params": {
      "output": ["triggerid", "description", "priority"],
      "filter": { "value": 1 },
      "sortfield": "priority",
      "sortorder": "DESC"
    },
    "id": 2
  }
  ```

### 3. Motor de Autocuración (Self-Healing)
El sistema puede configurarse para resolver incidencias comunes de forma automatizada:
- **Evento:** Zabbix detecta que una app de PM2 (ej: `elitepass-pos`) está inactiva y dispara la alerta.
- **Acción automática:** El webhook en `elitepass-monitor` procesa la alerta, identifica que la severidad es alta y la app es crítica, y ejecuta localmente `pm2 restart elitepass-pos`.
- **Notificación:** Envía un mensaje a Telegram indicando: `"⚠️ Incidencia detectada: elitepass-pos caído. 🚀 Intentando auto-recuperación local... ✅ Servicio restablecido exitosamente."`
- **Garantía de Seguridad:** Máximo de 3 reintentos automáticos en un intervalo de 10 minutos para evitar bucles infinitos en caso de fallos persistentes de código.
