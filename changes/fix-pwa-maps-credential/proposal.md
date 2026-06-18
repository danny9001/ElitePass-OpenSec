# Propuesta: Solución de Errores de Precarga PWA, Mapas de Reserva y Desfase de Credenciales

## 1. Intentos de Negocio / Reporte de Errores
El cliente reporta múltiples errores críticos en producción (`reservas.genial-it.net`) al usar la PWA instalada, especialmente al probar en un celular Samsung S7 Edge:

1. **Error de Precarga del Service Worker (`bad-precaching-response`):**
   - El Service Worker (`sw.js`) de Serwist intenta precargar un manifiesto de build anterior (`_buildManifest.js`), el cual ya no existe en el servidor (status 404), bloqueando la instalación de la PWA.
2. **Violación de Content Security Policy (CSP) en Navegación (Páginas en blanco):**
   - Al navegar a rutas como `/packages`, `/map-templates` y otras, el Service Worker antiguo o Next.js sirve una página HTML index cacheada (con nonces CSP obsoletos del build anterior). Esto genera un mismatch con el nonce CSP de la cabecera actual, causando que el navegador bloquee todos los scripts de hidratación de Next.js. Al no inicializarse React, la app queda en blanco y no funcionan elementos interactivos (como el menú de notificaciones).
3. **Error de MIME Type y Carga de Chunks:**
   - La página cacheada vieja intenta cargar scripts viejos (`webpack-b0f91ef.js`) que ya no existen en el servidor (dando 400/404). El servidor responde con HTML de error, lo que hace que el navegador rechace el script debido a que el tipo MIME no es ejecutable (`Refused to execute script ... MIME type ('text/html') is not executable`).
4. **Fallas al cargar mapas en la PWA (sólo se ven los objetos):**
   - Al instalar la PWA, la imagen de fondo del mapa no se visualiza. Sólo se muestran los objetos (mesas) en el canvas. Esto se debe a que el Service Worker intercepta la petición a la imagen como respuesta opaca o bloqueada por CORS en Canvas.
5. **Desfase de la imagen de fondo personalizada de la credencial en celulares antiguos:**
   - La imagen de fondo personalizada de la credencial (`credentialBgImageUrl`) se ve desfasada o no cubre toda la superficie en el Samsung S7 Edge. Esto se debe a un bug de renderizado clásico de texturas 3D (`background-size: cover` con transformaciones 3D/backface-visibility) en navegadores antiguos. Además, al usar CSS `backgroundImage`, la imagen de fondo no se descarga correctamente mediante `html-to-image` porque no se detecta en el query selector de imágenes para convertir a data URLs.

---

## 2. Solución Propuesta
1. **PWA Precaching Fix:** Configurar `@serwist/next` en `next.config.js` para excluir `_buildManifest.js` y `_ssgManifest.js` del precaching.
2. **CSP Nonce propagation:** Pasar el `nonce` generado al componente `ThemeProvider` en `layout.tsx` para evitar que el script de inicialización de tema viole la CSP.
3. **CORS & Canvas Image Fix:** Configurar la propiedad `crossOrigin = "anonymous"` en el objeto `Image` de carga en `table-layout-canvas.tsx` antes de asignar la URL de origen de la imagen de fondo, permitiendo que sea cacheada y leída correctamente por el Canvas sin causar bloqueos CORS.
4. **Credential Background Image Rendering & Alignment Fix:** Cambiar el uso de `backgroundImage` CSS en la credencial virtual por un tag `<img>` absoluto con `object-cover` y `-z-10`.
5. **Solución a Páginas en Blanco y Mismatch de Nonces:**
   - Modificar [sw.ts](file:///home/soporte/elitepass-reservas/src/app/sw.ts) para configurar `cleanupOutdatedCaches: true` en las opciones de precaché, asegurando que las cachés viejas se eliminen inmediatamente.
   - Forzar a que **todas** las peticiones de navegación (`request.mode === "navigate"`) usen la estrategia `NetworkOnly` en `sw.ts`. Esto garantiza que los documentos HTML dinámicos con CSP dinámico nunca se sirvan desde el caché (evitando mismatch de nonces), manteniendo el fallback offline para el `/offline.html` si no hay conexión.
