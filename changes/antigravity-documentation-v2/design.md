# Design: Estructura de la Documentación del Ecosistema e Integración de Mundial

## Decisiones Técnicas y de Ubicación
Para garantizar la persistencia del conocimiento de la arquitectura y que ambos agentes (Claude/Gemini) tengan acceso en tiempo real a la misma información, se mantendrá la estructura estandarizada en `/home/soporte/openspec/specs/antigravity/`.

Se agregará un nuevo archivo `/home/soporte/openspec/specs/antigravity/elitepass-mundial.md` para documentar la aplicación Mundial.

### Cambios Clave por Archivo

1. **`overview.md` (Topología e Integraciones):**
   - **Diagrama Mermaid:** Agregar `VM02-MUNDIAL` (Host 10.0.0.7) al diagrama de bloques y de flujo. Mostrar que Nginx rutea solicitudes a `mundial.genial-it.net` redirigiéndolas al puerto `3001` (Podman) de VM02.
   - **Flujo de Sincronización:** Documentar el flujo HTTP POST (`/api/v1/sync`) desde POS (`backend/src/config/identity-sync.ts`) y Reservas (`src/lib/identity-client.ts`) hacia Identity.
   - **Redirecciones SSO:** Documentar la lógica de redirección post-login: `EXTERNAL` → `/eventos`, Staff → `/dashboard`.

2. **`elitepass-reservas.md`:**
   - **Fidelización e Índices:** Documentar el índice compuesto `[userId, hasActivity]` para optimizar las consultas de `UserActivity`.
   - **Imágenes WebP:** Explicar el proceso de conversión automática de vouchers a WebP usando `sharp`.
   - **Facturación SIAT:** Detallar la lógica de selección de facturación por evento.

3. **`elitepass-pos.md`:**
   - **Sincronización:** Documentar el cliente de sincronización hacia Identity.
   - **Resiliencia Frontend:** Agregar la descripción del `ChunkErrorBoundary` en `main.tsx` que recarga la aplicación en caso de error por Service Worker desactualizado.

4. **`elitepass-payments.md`:**
   - **Configuración de Pasarelas:** Agregar los modos "ELITE" (compartido) y "CUSTOM" (por organización/evento).
   - **Compresión Nginx/Express:** Documentar la optimización de compresión con `compression` en Express a nivel de aplicación.

5. **`elitepass-identity.md`:**
   - **API Sync:** Documentar la estructura de la API `/api/v1/sync` (empresa, usuario, licencia).
   - **Fix de Redirect Loop:** Documentar la solución aplicada en `LoginForm.tsx` para evitar redirecciones prematuras durante el login simétrico.

6. **`elitepass-mundial.md` (Nuevo):**
   - **Core Técnico:** Next.js 16, pg nativo, sin ORM (SQL raw), uploads locales, sin Redis ni PgBouncer.
   - **Runtime Dual:** Explicar el clúster de Podman en el puerto 3001 ( nginx-lb -> app_1/app_2 ) y PM2 corriendo en puerto 3002.
   - **Deploy:** Detallar el flujo de comandos para compilar la imagen de Podman y reiniciar servicios.
