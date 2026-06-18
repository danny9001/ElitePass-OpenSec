# Diseño — Corrección de Autenticación, Monitoreo e Infraestructura 2026-06-09

## Soluciones Técnicas Aplicadas

### 1. Corrección de Redirecciones a Localhost
- **Problema:** En el cliente Next.js de `elitepass-identity`, la variable `NEXT_PUBLIC_APP_URL` no estaba compilada en la compilación de producción, causando un fallback a `http://localhost:3300`. Al iniciar sesión por correo o Google OAuth, la API redirigía incorrectamente al localhost local del usuario.
- **Solución:** Modificado [auth-client.ts](file:///home/soporte/elitepass-identity/src/lib/auth-client.ts) para usar `window.location.origin` de manera dinámica si se ejecuta en el navegador. También se configuraron explícitamente los dominios de confianza en [auth.ts](file:///home/soporte/elitepass-identity/src/lib/auth.ts).

### 2. Contraseñas y Roles de Super Admin
- Se configuró la contraseña `L4nd1v4r$$$` encriptada usando el algoritmo criptográfico de `better-auth` en la base de datos `elitepass_identity` para el usuario `dlandivar@genial-it.net` y se actualizó su rol a `superadmin` e ingresó como `SUPER_ADMIN` en `jet_club_db`.

### 3. Compresión Brotli en Nginx
- Se instalaron los módulos `libnginx-mod-http-brotli-filter` y `libnginx-mod-http-brotli-static`.
- Se configuró Brotli mediante el archivo `/etc/nginx/conf.d/brotli.conf` para evitar modificaciones directas en el core de `/etc/nginx/nginx.conf`.
- Se verificó y recargó la configuración de Nginx de manera segura.

### 4. Sincronización de Rate Limit (PM2 Cluster)
- **Problema:** Cada worker del clúster de Next.js tenía su propio almacén de rate limits en memoria, duplicando o haciendo inestable la limitación de peticiones.
- **Solución:** Se implementó `applyDbRateLimit` en [middleware.ts](file:///home/soporte/elitepass-reservas/src/middleware.ts), el cual realiza una petición HTTP interna y segura al endpoint `/api/internal/rate-limit`, centralizando el consumo de cuotas en la tabla `RateLimitBucket` de PostgreSQL. Posee un fallback automático en memoria si falla la base de datos.

### 5. Integración del Monitoreo
- Se registró el servicio `elitepass-identity` en el puerto 3300 en el script [monitor-helper.js](file:///home/soporte/monitor-helper.js), permitiendo al bot y a las herramientas de visualización monitorear su estado, uso de CPU y memoria en tiempo real.
