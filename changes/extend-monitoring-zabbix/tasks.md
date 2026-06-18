# Tasks: Extender el Sistema de Monitoreo con Zabbix

- [ ] **Fase 1: Preparación e Instalación en VM00**
  - [ ] Agregar el repositorio oficial de Zabbix para Ubuntu 24.04.
  - [ ] Instalar `zabbix-agent2` y sus utilidades básicas.
  - [ ] Agregar el usuario `zabbix` al grupo `docker` para permitir el monitoreo de contenedores.

- [ ] **Fase 2: Configuración del Agente y Base de Datos**
  - [ ] Crear el usuario `zbx_monitor` en PostgreSQL con permisos de `pg_monitor`.
  - [ ] Configurar el archivo de configuración del agente (`zabbix_agent2.conf`) con la IP del Zabbix Server.
  - [ ] Habilitar y configurar las variables de entorno de base de datos en `/etc/zabbix/zabbix_agent2.d/plugins.d/postgresql.conf`.
  - [ ] Agregar el archivo `/etc/zabbix/zabbix_agent2.d/pm2.conf` con los UserParameters para PM2.
  - [ ] Habilitar y reiniciar el servicio `zabbix-agent2`.

- [ ] **Fase 3: Configuración en Zabbix Server (Interfaz Web)**
  - [ ] Registrar el Host `VM00` en Zabbix Server usando la IP y puerto 10050.
  - [ ] Enlazar las plantillas base: `Linux by Zabbix Agent`, `Docker`, y `PostgreSQL by Zabbix Agent 2`.
  - [ ] Configurar los Escenarios Web (HTTP check) para:
    - [ ] `https://id.genial-it.net`
    - [ ] `https://reservas.genial-it.net`
    - [ ] `https://pos.genial-it.net`
  - [ ] Configurar los triggers para latencia alta (>2s) y expiración de SSL (<10 días).

- [ ] **Fase 4: Alertas e Integración de Notificaciones**
  - [ ] Configurar el Media Type `Telegram` en Zabbix Server con el token del Bot actual.
  - [ ] Crear la acción de alertas para enviar mensajes ante problemas críticos al chat de soporte.
  - [ ] Validar el envío de una alerta de prueba.
