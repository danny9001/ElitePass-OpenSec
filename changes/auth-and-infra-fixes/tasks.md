# Tasks — Corrección de Autenticación, Monitoreo e Infraestructura 2026-06-09

- [x] Redirección de Autenticación (BetterAuth): Corregido `baseURL` en `elitepass-identity/src/lib/auth-client.ts` para usar dinámicamente `window.location.origin` y evitar bucles de redirección a `localhost`.
- [x] Configuración de Super Admins: Modificadas las credenciales de `dlandivar@genial-it.net` en `elitepass_identity` y `jet_club_db` estableciendo el rol correcto (superadmin/SUPER_ADMIN) y la contraseña encriptada `L4nd1v4r$$$`.
- [x] Instalación de Brotli en Nginx: Instalado `libnginx-mod-http-brotli-filter` y `libnginx-mod-http-brotli-static` y configurado en `/etc/nginx/conf.d/brotli.conf`.
- [x] Duplicación de Rate Limit: Centralizado el rate limiting de `elitepass-reservas` a nivel de base de datos utilizando la tabla `RateLimitBucket` y el endpoint `/api/internal/rate-limit`.
- [x] Monitoreo: Registrado `elitepass-identity` en el puerto 3300 en el script `/home/soporte/monitor-helper.js`.
- [x] Compilar, Commit, Push y Deploy: Realizado el build en producción, guardado en git, empujado al remote `danny` y deployado en PM2.
