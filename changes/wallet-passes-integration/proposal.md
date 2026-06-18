# Proposal: Wallet Passes Integration (Google & Apple Wallet)

## Business Intent
Introduce native integration with Apple Wallet (iOS) and Google Wallet (Android) for ElitePass digital credentials. Clients (VIP, internal, external) should be able to add their custom passes directly to their device's native wallet app. 
This provides a premium offline experience where users can:
- Present their digital credential at local gates.
- Render their custom credential design, VIP/Role badges, and affiliated companies.
- Access their Universal QR code for quick scanning without opening the browser.
- Receive push updates and geolocated lock screen notifications when near an affiliated club/event.

## Goal
Implement a standalone microservice named `elitepass-billeteraQR` (or `elitepass-wallet`) that manages:
1. Generation and signing of Apple Wallet `.pkpass` files.
2. Building and signing JWT URLs for Google Wallet passes.
3. Exposing a secure internal API to allow `elitepass-reservas` and other apps to request pass assets.

## Scope & Target Audience
- **Apple Wallet:** iOS native integration using `.pkpass` format (requires Apple Developer certificate for signing).
- **Google Wallet:** Android native integration using Google Wallet REST API and JWT generation (requires Google Pay Developer console credentials).
- **Microservice:** `elitepass-billeteraQR` built using Node.js/TypeScript to isolate signing overhead and certificate storage.
- **Client Integration:** Add "Add to Apple Wallet" and "Add to Google Wallet" buttons to the client profile/membership views.

## Restricciones de Credenciales y Estrategia
Al no contar con cuentas de desarrollador activas de Apple ni Google en este momento, aplicamos la siguiente estrategia:

1. **Simulación en Desarrollo (Mocking):** El microservicio `elitepass-billeteraQR` se implementará con respuestas simuladas (stubs) y certificados auto-firmados de prueba. Esto deja la arquitectura de código 100% lista para que, cuando el cliente adquiera sus cuentas de desarrollador, solo deba colocar los certificados reales en `/certs` y activar el servicio sin tocar una línea de código.
2. **Solución Alternativa Inmediata (PWA Virtual Pass):**
   - El ecosistema ya funciona como una PWA (Progressive Web App).
   - Desarrollaremos/optimizaremos una **Credencial Virtual PWA** interactiva y offline dentro de la misma aplicación, con diseño premium (estilo acromático zinc), que imita el aspecto de una tarjeta de Wallet, mostrando su QR Universal, rol (VIP, etc.) y empresas.
   - Esta solución tiene costo $0 y funciona inmediatamente en cualquier dispositivo móvil sin necesidad de cuentas externas.

