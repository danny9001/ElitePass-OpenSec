# Proposal: Sistema Híbrido de Monitoreo y ChatOps (Zabbix + ElitePass Monitor)

## Business Intent
El objetivo es fusionar las capacidades analíticas y de persistencia de **Zabbix** con la interfaz conversacional e interactiva de **ElitePass Monitor (Telegram Bot)**. Esto dará lugar a un nuevo sistema unificado de **ChatOps y Autocuración (Self-Healing)**. Zabbix se encargará del procesamiento de métricas, disparo de alarmas complejas e historización, mientras que el Bot actuará como la interfaz de control (visualización de gráficos históricos, reconocimiento de alarmas y reinicios rápidos) y como agente ejecutor de auto-recuperación ante eventos críticos.

## Scope
1. **Integración de Zabbix API en el Bot:** Habilitar al Bot para interactuar con la API JSON-RPC de Zabbix Server, permitiendo consultas de estado como `/alertas` (triggers activos) y `/grafico <item>` (retornando una imagen del gráfico de métrica generada por Zabbix).
2. **Alertas Ricas con Botones Interactivos (Webhooks):** Extender `elitepass-monitor` para exponer un puerto/endpoint seguro (ej: `/api/v1/zabbix-alert`) que reciba las notificaciones de Zabbix Server. El bot enviará las alertas a Telegram formateadas en HTML enriquecido con botones inline (ej: `[Reconocer Alerta]`, `[Ver logs de PM2]`, `[Reiniciar Proceso]`).
3. **Autocuración Automatizada (Self-Healing):** Programar el bot para recibir señales de caída de Zabbix y ejecutar automáticamente acciones correctivas locales (ej. reiniciar un worker pm2 caído, liberar pools de postgres, limpiar memoria cache de redis) y reportar la ejecución inmediata al chat.
