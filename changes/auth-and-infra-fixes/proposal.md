# Propuesta — Corrección de Autenticación, Monitoreo e Infraestructura 2026-06-09

## Business Intent
Resolver los fallos de autenticación centralizada en el microservicio `elitepass-identity` (redirecciones incorrectas a `localhost` y fallos en Google OAuth), configurar las credenciales obligatorias para los administradores del sistema, habilitar la compresión Brotli en el servidor web Nginx, resolver la duplicación de rate limits en los procesos de clúster de Next.js, y registrar `elitepass-identity` en el panel de monitoreo.

## Scope de la Solución
1. **Redirección de Autenticación (BetterAuth):**
   - Corregir variables de entorno en `elitepass-identity` (ej. `BETTER_AUTH_URL` o configuraciones de URL base) que causan redirección a `localhost`.
   - Ajustar el flujo de Google OAuth para asegurar el retorno correcto de la sesión sin rebotar al login.
2. **Super Admins:**
   - Asegurar que `dlandivar@genial-it.net` tenga el rol `SUPER_ADMIN` y su contraseña establecida como `L4nd1v4r$$$` en la base de datos de identidad / reservas.
3. **Instalación de Brotli en Nginx:**
   - Instalar `libnginx-mod-brotli`.
   - Modificar `/etc/nginx/nginx.conf` o el bloque de host virtual local para habilitar y configurar la compresión Brotli.
4. **Duplicación de Rate Limit:**
   - Implementar la limitación a nivel de Nginx para los endpoints dinámicos críticos (como `/api/auth/` o Server Actions).
5. **Monitoreo:**
   - Añadir la validación del microservicio `elitepass-identity` (puerto 3300) en el panel [monitor-helper.js](file:///home/soporte/monitor-helper.js).
