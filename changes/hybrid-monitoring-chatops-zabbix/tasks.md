# Tasks: Sistema Híbrido de Monitoreo y ChatOps (Zabbix + ElitePass Monitor)

- [ ] **Fase 1: Servidor de Enlace en `elitepass-monitor`**
  - [ ] Añadir soporte para servidor HTTP en `src/index.ts` o crear un nuevo archivo de escucha.
  - [ ] Configurar puerto de escucha (ej. 3250) y proteger el endpoint con un token de portador (`Bearer Token`).
  - [ ] Implementar la ruta `POST /api/v1/zabbix-alert`.

- [ ] **Fase 2: Mensajería Interactiva de Alertas**
  - [ ] Diseñar plantilla HTML en el bot para mostrar alertas enriquecidas (iconos de severidad, host, trigger, valor).
  - [ ] Implementar la construcción de botones inline de Telegram (`InlineKeyboardMarkup`):
    - [ ] Botón "Reconocer en Zabbix"
    - [ ] Botón "Reiniciar Servicio"
    - [ ] Botón "Ver logs de PM2"
  - [ ] Programar los controladores de callback query del bot para recibir la respuesta al hacer clic en los botones.

- [ ] **Fase 3: Cliente API JSON-RPC de Zabbix**
  - [ ] Implementar el cliente para autenticarse y obtener un token de sesión desde Zabbix Server.
  - [ ] Implementar el método para marcar un evento/incidencia como reconocido (`event.acknowledge`).
  - [ ] Implementar el comando de consulta `/alertas` que interactúa con la API de Zabbix para recuperar triggers activos.

- [ ] **Fase 4: Motor de Autocuración (Self-Healing)**
  - [ ] Crear la lógica para identificar triggers automáticos remediables (ej: "Process offline").
  - [ ] Implementar ejecución segura de comandos locales (ej: `pm2 restart <nombre_app>`).
  - [ ] Agregar límites y protecciones: control de reintentos máximos (ej: no más de 3 reinicios en 10 min) para evitar loops ante bugs persistentes.

- [ ] **Fase 5: Configuración en Zabbix Server**
  - [ ] Registrar un nuevo "Media Type" de tipo Webhook en Zabbix Server.
  - [ ] Escribir el script JS en Zabbix para empaquetar los macros del trigger (`{EVENT.ID}`, `{TRIGGER.NAME}`, `{TRIGGER.SEVERITY}`, etc.) y hacer la petición POST a `https://id.genial-it.net:3250/api/v1/zabbix-alert`.
  - [ ] Crear la regla de acción en Zabbix para despachar la notificación a través de este canal ante cualquier trigger crítico.
