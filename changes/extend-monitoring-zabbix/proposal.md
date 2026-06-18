# Proposal: Extender el Sistema de Monitoreo con Zabbix

## Business Intent
El objetivo es complementar el sistema de monitoreo básico actual (`elitepass-monitor`) con **Zabbix**, una solución de monitoreo de grado empresarial de código abierto. Esto permitirá obtener métricas históricas, visualizaciones de rendimiento (dashboards), análisis de tendencias, monitoreo detallado de base de datos (PostgreSQL/PgBouncer) y alertas avanzadas (incluyendo vencimiento de certificados SSL y latencia de red) sin alterar ni retirar la funcionalidad de alertas rápidas de Telegram que posee el script actual.

## Scope
1. **Arquitectura No Intrusiva:** Diseñar la coexistencia de `elitepass-monitor` (alertas instantáneas a Telegram) y Zabbix Agent 2 (monitoreo profundo y analíticas) en el servidor `VM00`.
2. **Monitoreo de Infraestructura:** Configurar Zabbix Agent 2 para recopilar métricas de CPU, RAM, almacenamiento, servicios systemd, estado de Docker y PM2.
3. **Monitoreo de Base de Datos:** Integrar el plugin nativo de Zabbix para PostgreSQL, monitoreando métricas de rendimiento, pools de conexión y salud general de la base de datos a través de PgBouncer.
4. **Synthetic Web Monitoring:** Crear escenarios web en Zabbix Server para auditar la disponibilidad, latencia y expiración de certificados SSL de `https://id.genial-it.net`, `https://reservas.genial-it.net` y `https://pos.genial-it.net`.
