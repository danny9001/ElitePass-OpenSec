# Proposal: Actualización de Documentación Antigravity e Integración de Mundial

## Business Intent
El ecosistema de aplicaciones **ElitePass / Antigravity** ha evolucionado con importantes mejoras recientes en autenticación única (SSO), sincronización automática de datos entre apps satélites e Identity, optimizaciones de Nginx (Brotli, rate-limits compartidos, eliminación de headers duplicados), y optimizaciones de base de datos (índice en `UserActivity` y WebP). Además, la aplicación **apuestas-mundial-2026** (desplegada en VM02-MUNDIAL) no cuenta con documentación en el repositorio global de especificaciones de la arquitectura Antigravity.

El objetivo de esta propuesta es actualizar la documentación existente para reflejar fielmente el estado actual del ecosistema y generar la especificación completa de la aplicación Mundial en `openspec/specs/antigravity/`.

## Scope
1. **Actualizar overview.md:**
   - Incorporar la aplicación satélite **apuestas-mundial-2026** en la topología de red y diagramas globales.
   - Documentar la arquitectura de sincronización de apps satélite hacia Identity (`/api/v1/sync`).
   - Actualizar el flujo de callbacks, mapeo de roles y redirecciones SSO post-autenticación (EXTERNAL vs Staff).
2. **Actualizar especificaciones individuales:**
   - `elitepass-reservas.md`: Documentar el cliente de sincronización con Identity, el índice `[userId, hasActivity]`, WebP automático y la facturación SIAT.
   - `elitepass-pos.md`: Documentar la sincronización de usuarios/empresas con Identity y el fix del bundle (`ChunkErrorBoundary`).
   - `elitepass-payments.md`: Incluir la optimización de compresión con Express compression gzip y el toggle de pasarelas "ELITE" vs "CUSTOM".
   - `elitepass-identity.md`: Detallar la nueva API idempotente `/api/v1/sync` y el flujo de autenticación local vs SSO (evitando redirect loops).
3. **Crear especificación elitepass-mundial.md:**
   - Documentar la arquitectura del sistema apuestas-mundial en `VM02-MUNDIAL`.
   - Detallar su stack de runtime dual (Podman para tráfico real en 3001 y PM2 en 3002).
   - Documentar el esquema de base de datos PostgreSQL local en contenedor y el uso de `pg` nativo (sin Prisma).
   - Documentar la autenticación local (WebAuthn + JWT) y el scheduler de PM2.
