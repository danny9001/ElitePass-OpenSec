# Lista de Tareas para la Corrección de PWA, CSP y Credencial

- [x] **Paso 1: Excluir manifiestos del precaching de Serwist**
  - Modificar [next.config.js](file:///home/soporte/elitepass-reservas/next.config.js) para añadir la opción `exclude` en la configuración de `withSerwistInit`.
  - Excluir archivos que coincidan con `/.*buildManifest.*\.js$/` y `/.*ssgManifest.*\.js$/`.

- [x] **Paso 2: Pasar CSP Nonce a ThemeProvider**
  - Modificar [src/app/layout.tsx](file:///home/soporte/elitepass-reservas/src/app/layout.tsx) para pasar el prop `nonce={nonce}` al componente `<ThemeProvider>`.

- [x] **Paso 3: Añadir crossOrigin condicional a la imagen de fondo del Canvas**
  - Modificar [src/components/system/map-templates/table-layout-canvas.tsx](file:///home/soporte/elitepass-reservas/src/components/system/map-templates/table-layout-canvas.tsx) para definir `img.crossOrigin = "anonymous";` solo cuando la URL sea externa (`http/https`) y no pertenezca al mismo origen (`window.location.origin`). Esto evita bloqueos CORS ficticios en archivos locales de ejemplo.

- [x] **Paso 4: Corregir posicionamiento del logo y renderizado de la imagen de fondo en la Credencial Virtual**
  - Modificar [src/components/shared/virtual-credential/virtual-credential-dialog.tsx](file:///home/soporte/elitepass-reservas/src/components/shared/virtual-credential/virtual-credential-dialog.tsx).
  - Mover el contenedor de "Company Logo Overlay" dentro del contenedor de la foto de perfil (`div` de la foto de perfil `w-32 h-32`). (Completado)
  - Cambiar el posicionamiento del overlay de logo a `bottom-0 right-0 z-20` relativo a la foto de perfil. (Completado)
  - Reemplazar el uso de `backgroundImage` en CSS en los divs del Frontal y Reverso de la tarjeta por una etiqueta HTML `<img>` con `absolute inset-0 w-full h-full object-cover pointer-events-none -z-10`, dejando el gradiente como color de fondo base (fallback) en el div.

- [x] **Paso 4.6: Revertir sw.ts y restaurar backgroundImage en Credencial**
  - Revertir los cambios locales de [sw.ts](file:///home/soporte/elitepass-reservas/src/app/sw.ts) al estado original de git. (Completado)
  - Revertir la imagen de fondo en [virtual-credential-dialog.tsx](file:///home/soporte/elitepass-reservas/src/components/shared/virtual-credential/virtual-credential-dialog.tsx) al uso de `backgroundImage` CSS. (Completado)

- [x] **Paso 4.7: Corregir Hydration Mismatch en Credencial**
  - Modificar [virtual-credential-dialog.tsx](file:///home/soporte/elitepass-reservas/src/components/shared/virtual-credential/virtual-credential-dialog.tsx) para inicializar `data` como `null` en `useState` y cargar el `localStorage` en un `useEffect` ejecutado en el cliente. Esto corregirá el error de hidratación de React y restaurará el menú superior y los dropdowns. (Completado)

- [x] **Paso 4.8: Añadir hash SHA-256 de next-themes a middleware.ts**
  - Modificar [src/middleware.ts](file:///home/soporte/elitepass-reservas/src/middleware.ts) para agregar `'sha256-J9cZHZf5nVZbsm7Pqxc8RsURv1AIXkMgbhfrZvoOs/A='` en la sección `script-src` de la política CSP. (Completado)

- [x] **Paso 4.9: Ejecutar despliegue completo mediante deploy.sh**
  - Ejecutar [deploy.sh](file:///home/soporte/elitepass-reservas/deploy.sh) para recompilar la aplicación, recargar los procesos en caliente de PM2 (`pm2 reload`) y purgar la caché de Cloudflare. (Completado)

- [ ] **Paso 5: Probar y Verificar compilación**
  - Ejecutar tests o builds de producción local para verificar que compile correctamente.
  - Asegurar la actualización del versionado en `package.json` siguiendo el versionado PATCH +1 como indica `skill-elitepass-organizer.md`.
