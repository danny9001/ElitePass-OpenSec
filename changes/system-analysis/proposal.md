# Análisis del Ecosistema ElitePass — 2026-06-09

## Business Intent

Realizar una auditoría y análisis técnico exhaustivo de los seis sistemas que componen el ecosistema de **ElitePass** (Reservas, POS, Pagos, Identidad, Notificaciones y Monitoreo), detallando su topología de red, puertos de escucha, interacción de bases de datos, flujos de integración y mecanismos de seguridad aplicados. Este análisis servirá como fuente de verdad única para el mantenimiento preventivo y futuros desarrollos en el servidor VM00-Reservas-v1.

## Scope del Análisis

1. **Topología de Sistemas y Puertos:** Detallar cada una de las 6 aplicaciones/microservicios, sus puertos, tecnologías y procesos en PM2.
2. **Esquema de Bases de Datos:** Identificar las bases de datos y esquemas de PostgreSQL a los que se conecta cada microservicio.
3. **Flujos de Interacción:** Mapear cómo se comunican las aplicaciones entre sí (ej. pasarelas de pago, callbacks con HMAC, notificaciones Telegram).
4. **Mecanismos de Seguridad:** Identificar los esquemas de autenticación (WebAuthn/BetterAuth/JWT) y protección (WAF/WIP, Rate Limiting, cifrado AES-256).
5. **Gaps y Oportunidades de Mejora:** Identificar inconsistencias en monitorización, seguridad o rendimiento para proponer soluciones.
