# Proposal: Documentación de la Arquitectura "Antigravity" (Ecosistema ElitePass)

## Business Intent
El objetivo es documentar a fondo y estructurar la arquitectura del ecosistema de sistemas de Genial-it / ElitePass bajo el nombre clave y concepto de **"Antigravity"**. Se creará un documento Markdown (.md) detallado para cada uno de los 6 sistemas satélites y un documento integrador de relaciones, siguiendo la filosofía de diseño "zero-gravity" (alta disponibilidad, baja latencia, componentes altamente desacoplados y libre de fricción operativa).

## Scope
1. **Definición del Ecosistema Antigravity:** Unificar los sistemas existentes bajo la metáfora de gravedad cero (componentes ultraligeros, colas asíncronas con reintentos para evitar fricción, etc.).
2. **Generación de 7 Archivos Markdown de Documentación:**
   - `overview.md`: Relación y topología global del ecosistema, flujos de datos e interacciones (pagos, notificaciones, auth).
   - `elitepass-reservas.md`: El sistema central de reservas y eventos (Next.js 15).
   - `elitepass-pos.md`: El ERP y Punto de Venta para barras y cajas (Express + React).
   - `elitepass-payments.md`: La pasarela descentralizada de pagos QR BCP/BNB y SIAT.
   - `elitepass-identity.md`: El proveedor SSO central con passkeys y BetterAuth.
   - `elitepass-noti-telegram.md`: El microservicio de mensajería asíncrona.
   - `elitepass-monitor.md`: El agente de salud del servidor y monitoreo SSH/CrowdSec.
3. **Uso de Mermaid.js:** Integración de diagramas de bloques, flujo y ERD para dar soporte visual a cada sistema.
