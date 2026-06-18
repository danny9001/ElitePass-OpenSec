# Proposal: Mezclar y Unificar elitepass-monitor con Zabbix (Monitoreo Nativo Trapper)

## Business Intent
El objetivo es fusionar **elitepass-monitor** y **Zabbix** en una única solución unificada en lugar de ejecutarlos como sistemas independientes. En este modelo integrado, **elitepass-monitor** actuará como el agente recopilador y despachador oficial para Zabbix Server. El bot recopilará las métricas del servidor (CPU, RAM, Disco, Docker, PM2) y, al mismo tiempo que las evalúa para enviar alertas interactivas a Telegram, las transmitirá directamente a Zabbix Server utilizando el protocolo nativo TCP Trapper (Zabbix Sender). Esto elimina la necesidad de instalar o mantener un daemon Zabbix Agent independiente en el servidor, unificando la lógica de recolección en un solo codebase.

## Scope
1. **Zabbix Sender Integrado en el Bot:** Desarrollar un cliente TCP ligero en TypeScript dentro de `elitepass-monitor` que implemente el protocolo de cabeceras de Zabbix (`ZBXD\x01`) para enviar métricas directamente al puerto `10051` del Zabbix Server.
2. **Eliminación del Agente Externo:** Usar la lógica de recolección de `MonitorService` para alimentar a Zabbix Server en cada ciclo de Health Check, garantizando que Zabbix y el Bot compartan exactamente los mismos datos de telemetría y estado.
3. **Consolidación de Configuración:** Centralizar las variables del servidor Zabbix (`ZABBIX_SERVER_IP`, `ZABBIX_PORT`, `ZABBIX_HOST_NAME`) en el archivo `.env` de `elitepass-monitor`.
