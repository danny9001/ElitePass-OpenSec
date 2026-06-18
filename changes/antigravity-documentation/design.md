# Design: Estructuración y Enrutamiento de la Documentación Antigravity

## Decisiones Técnicas y de Ubicación
Para mantener la memoria del proyecto ordenada y accesible tanto para Claude como para Gemini/Antigravity (de acuerdo con `RULE[AGENTS.md]`), los entregables se guardarán en una carpeta dedicada bajo la memoria global:
`~/openspec/specs/antigravity/` (es decir, `/home/soporte/openspec/specs/antigravity/`).

### Estructura de Documentación de cada Sistema (Estructura "Antigravity")
Cada documento de sistema individual seguirá este formato:
1. **Core Técnico y Arquitectura:** Rol del sistema, stack tecnológico específico y diagrama de flujo interno.
2. **Capa de Datos y Persistencia:** Esquema de base de datos e índices clave (o uso de memoria/OS si aplica).
3. **Mecanismos de Seguridad e Hardening:** Autenticación, control de accesos (roles) y rate limits.
4. **Despliegue e Infraestructura:** Proceso en PM2/Docker, puerto de escucha y variables críticas de entorno.

### Relación Global (overview.md)
Detallará:
- Diagrama de flujo general de solicitudes (Topología de Red) usando Mermaid.js.
- Patrón arquitectónico (Monolito modular interactuando con microservicios asíncronos y SSO federado).
- Detalle del flujo de autenticación (SSO Identity Callback), flujo de pagos (Webhook -> HMAC Callback -> SIAT), y flujo de notificaciones (Cola persistente Redis -> Telegram REST).
