# Diseño Técnico: Solución de Errores de PWA, CSP y Renderizado de Mapas/Credenciales

## 1. Análisis de Causa Raíz

### 1.1 Error de Precaching (Serwist 404)
- **Causa:** En cada compilación de Next.js, se generan nuevos archivos con hash, incluyendo `_buildManifest.js` y `_ssgManifest.js`. Serwist intenta precargar estos archivos por defecto. Si un usuario tiene un Service Worker activo anterior, al cargarse una versión nueva, el SW intenta precargar el `_buildManifest.js` viejo, el cual ya fue borrado en producción (dando 404). Esto dispara el error `bad-precaching-response` de Workbox.
- **Resolución:** Excluir explícitamente `*buildManifest*.js` y `*ssgManifest*.js` de la lista de precaché en `next.config.js`.

### 1.2 Violación de CSP (Inline Script)
- **Causa:** El `ThemeProvider` (de `next-themes`) inyecta un script inline síncrono en el `<head>` para evitar parpadeos de tema oscuro/claro. Dado que `layout.tsx` no pasa el `nonce` generado por el middleware a `ThemeProvider`, este script inline no recibe el atributo `nonce="..."` en su renderizado, violando la política `script-src 'self' 'nonce-...'`.
- **Resolución:** Pasar `nonce={nonce}` a la etiqueta `<ThemeProvider>` en `src/app/layout.tsx`.

### 1.3 Canvas sin Imagen de Fondo (CORS / PWA)
- **Causa:** 
  1. Si las imágenes del mapa se cargan desde un CDN externo (como Azure Blob), el Canvas requiere que la imagen se solicite con CORS (`crossOrigin = "anonymous"`) para poder dibujarla sin violar la seguridad del Canvas.
  2. Sin embargo, si la imagen es del mismo origen (ruta relativa `/uploads/...` o del mismo host), el servidor Next.js sirve el archivo local sin enviar cabeceras CORS. Si se le define `crossOrigin = "anonymous"`, el navegador interpreta que el recurso requiere CORS, pero al no recibir las cabeceras de respuesta correspondientes, lo bloquea por seguridad. Esto causaba que las imágenes de mapa locales o de ejemplo fallaran al cargarse en PC y móviles.
- **Resolución:** Configurar de forma condicional `img.crossOrigin = "anonymous"` en [table-layout-canvas.tsx](file:///home/soporte/elitepass-reservas/src/components/system/map-templates/table-layout-canvas.tsx). Solo se aplicará el atributo si la URL es externa (`http/https`) y no pertenece al mismo origen (`window.location.origin`).


### 1.6 Acumulación de Caché Obsoleta (PWA Zombie)
- **Causa:** Cuando el Service Worker viejo falla su instalación (debido al error 404 de precacheo de `_buildManifest.js`), queda atrapado en un estado de error, y sigue sirviendo recursos obsoletos y corruptos desde la caché de la versión anterior.
- **Resolución:** Habilitar `cleanupOutdatedCaches: true` en las opciones de precaché de Serwist (`precacheOptions`) en `src/app/sw.ts` para indicarle al nuevo Service Worker que debe purgar e invalidar agresivamente todas las cachés y archivos de precarga de compilaciones anteriores.


### 1.7 Fallo Generalizado de Hidratación de React (Layout Roto)
- **Causa:** La credencial virtual (`VirtualCredentialDialog`) se inicializa leyendo del `localStorage` del navegador. En el servidor, el renderizado inicial es con `data = null` (sin etiqueta `<img>` o ciertos estilos), mientras que en el cliente se renderiza con los datos cacheados (con etiqueta `<img>`). Al insertar dinámicamente un nodo del DOM (`<img>`) durante la hidratación inicial, React lanza un error catastrófico de hidratación. Al ocurrir esto en el layout principal de la barra de navegación superior, React rompe la interactividad de toda la aplicación (dropdowns de perfil, cerrar sesión, notificaciones, etc.).
- **Resolución (Reversión Segura):**
  - Revertir los cambios locales de `src/app/sw.ts` al estado original para evitar cualquier interferencia en la navegación.
  - Revertir el renderizado de la imagen de fondo en `virtual-credential-dialog.tsx` al uso de la propiedad CSS `backgroundImage` que tolera diferencias de estilo sin romper el árbol del DOM.
  - Corregir el estado de hidratación inicial de `virtual-credential-dialog.tsx` inicializando `data` como `null` en el constructor del estado y cargando `localStorage` únicamente en el ciclo `useEffect` en el cliente. Esto garantiza que el HTML del servidor y el renderizado inicial del cliente coincidan exactamente (sin mismatch de hidratación).
  - Mantener las correcciones estructurales del logo de la empresa y el canvas que ya funcionaban.

### 1.8 Violación de CSP por Script de Tema (next-themes)
- **Causa:** `next-themes` inyecta un script inline síncrono al principio del layout para evitar parpadeos visuales al determinar el tema del sistema. Aunque se pase `nonce={nonce}` en el Server Component, en producción y bajo ciertos estados de caché Edge (Cloudflare), el `nonce` puede ser resuelto como `undefined` o no propagarse adecuadamente a la hidratación del cliente, provocando que la etiqueta `<script>` carezca del atributo `nonce="..."` y sea bloqueada por el navegador.
- **Resolución:** Autorizar explícitamente el script inline de `next-themes` agregando su firma digital SHA-256 (`'sha256-J9cZHZf5nVZbsm7Pqxc8RsURv1AIXkMgbhfrZvoOs/A='`) en la directiva `script-src` del CSP en `src/middleware.ts`. Esto permite la ejecución síncrona del tema de forma garantizada y sin importar si el nonce dinámico está disponible o no en esa petición concreta.

### 1.9 Desajuste de Assets en Caliente (ERR_ABORTED 400 en Chunks)
- **Causa:** Al ejecutar `pnpm run build` directamente, se generan nuevos archivos con hash en el disco, pero los procesos activos en memoria de PM2 (`elitepass-reservas`) siguen ejecutando el código del build viejo. Esta discrepancia entre la memoria y el disco hace que las peticiones del cliente a los nuevos chunks de Webpack devuelvan 400 Bad Request o 404, rompiendo la hidratación completa del frontend.
- **Resolución:** Ejecutar el flujo de despliegue oficial `./deploy.sh`, el cual compila, recarga los procesos de PM2 en caliente (`pm2 reload`) para sincronizar la memoria con el disco, y purga automáticamente la caché perimetral de Cloudflare.
