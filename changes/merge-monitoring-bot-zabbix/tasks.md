# Tasks: Mezclar y Unificar elitepass-monitor con Zabbix (Monitoreo Nativo Trapper)

- [ ] **Fase 1: Cliente TCP Zabbix Trapper**
  - [ ] Crear el archivo `src/zabbix-sender.ts` e implementar la función `sendToZabbix` con empaquetado binario (`ZBXD\x01` + length 64-bit LE + JSON payload).
  - [ ] Probar la conexión TCP de forma independiente con un script temporal de prueba.

- [ ] **Fase 2: Recolección y Despacho en `MonitorService`**
  - [ ] Implementar el método `pushMetricsToZabbix()` en `src/monitor-service.ts`.
  - [ ] Extraer los valores en tiempo real de:
    - [ ] Uso de CPU
    - [ ] Uso de RAM (porcentaje)
    - [ ] Uso de Disco (porcentaje)
    - [ ] Procesos de PM2 (totales y caídos)
    - [ ] Contenedores Docker (totales y detenidos)
  - [ ] Mapear los datos recolectados al formato de métricas de Zabbix.

- [ ] **Fase 3: Configuración y Ciclo Cron**
  - [ ] Añadir las variables de configuración en `/home/soporte/elitepass-monitor/.env`:
    - [ ] `ZABBIX_SERVER_IP`
    - [ ] `ZABBIX_PORT` (por defecto `10051`)
    - [ ] `ZABBIX_HOST_NAME` (nombre de host en Zabbix, ej. `VM00-Reservas-v2`)
  - [ ] Modificar `src/index.ts` para invocar `pushMetricsToZabbix()` de forma asíncrona dentro del intervalo del Health Check (cron).

- [ ] **Fase 4: Verificación y Despliegue**
  - [ ] Compilar y verificar el funcionamiento local en el monitor.
  - [ ] Reiniciar el proceso PM2 `elitepass-monitor`.
  - [ ] Verificar la recepción de métricas en Zabbix Server.
