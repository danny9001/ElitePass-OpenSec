# Especificación Funcional: Credencial Virtual PWA, PWA Install Button y Passkey Onboarding Prompt

## Contexto de Negocio
Para simplificar la instalación y el acceso sin contraseña (Passkeys/WebAuthn), se integran mecanismos intuitivos en el frontend. Los usuarios podrán instalar la aplicación mediante un botón explícito y configurar el inicio de sesión seguro biométrico inmediatamente después de autenticarse con sus credenciales estándar por primera vez.

---

## Escenarios de Comportamiento (BDD)

### Escenario 1: Inicialización rápida y soporte Offline (Stale-While-Revalidate)
* **GIVEN** que un cliente previamente autenticado abre la aplicación en su dispositivo sin conexión a internet (modo offline).
* **WHEN** abre el modal de su "Credencial Virtual" (pulsando el botón QR/tarjeta).
* **THEN** el sistema debe cargar inmediatamente los datos de la credencial guardados en el almacenamiento local (`localStorage`).
* **AND** debe generar y renderizar el código QR Universal de forma local y offline.
* **AND** debe intentar actualizar silenciosamente los datos desde el servidor en segundo plano si la conexión regresa.

### Escenario 2: Interacción 3D Flip Card
* **GIVEN** que el modal de la Credencial Virtual está abierto y mostrando el frente de la tarjeta.
* **WHEN** el usuario hace clic o pulsa en cualquier parte de la tarjeta.
* **THEN** la tarjeta debe realizar una animación tridimensional de volteo de 180 grados en el eje Y (`rotateY(180deg)`).
* **AND** debe revelar la parte trasera de la tarjeta, la cual contiene el QR Universal, el ID del cliente y la información de verificación sin conexión.
* **AND** al volver a pulsarla, debe rotar de regreso al frente.

### Escenario 3: Caché de Recursos Visuales
* **GIVEN** que el Service Worker de la PWA está activo.
* **WHEN** el usuario carga su perfil por primera vez con conexión a internet.
* **THEN** las imágenes asociadas al avatar de perfil y el logo de la organización deben ser almacenadas automáticamente en la caché local de imágenes de la PWA.
* **AND** cuando el usuario esté offline, estas imágenes deben seguir cargando correctamente en la credencial virtual.

### Escenario 4: Descarga del Badge / Tarjeta
* **GIVEN** que la Credencial Virtual está abierta.
* **WHEN** el usuario pulsa en el botón "Descargar".
* **THEN** el sistema debe renderizar y descargar una imagen limpia (.png) del frente de la credencial, independientemente de qué lado de la tarjeta esté visible en ese momento.

### Escenario 5: Botón de Instalación PWA (Home Page)
* **GIVEN** que un usuario accede al sitio web en un navegador que soporta PWA (ej. Chrome, Edge, Safari).
* **WHEN** la aplicación no está instalada en el dispositivo del usuario.
* **THEN** el sistema debe mostrar un botón con el texto "Instalar Aplicación" en la página de inicio.
* **AND** al pulsar este botón, debe dispararse el diálogo de instalación nativa del navegador.
* **AND** si la aplicación ya está instalada o el navegador no la soporta, el botón debe ocultarse automáticamente.

### Escenario 6: Oferta de Acceso sin Contraseña (Passkeys)
* **GIVEN** que un usuario inicia sesión por primera vez con su correo y contraseña (o no tiene llaves de acceso registradas).
* **WHEN** completa con éxito la autenticación y es redirigido a la aplicación.
* **THEN** el sistema debe verificar en segundo plano que el usuario tiene `0` passkeys asociadas.
* **AND** debe mostrar un diálogo modal ofreciendo configurar el acceso seguro con huella digital o rostro (Passkey).
* **AND** si el usuario acepta, debe iniciar el proceso biométrico de registro nativo.
* **AND** si el usuario lo descarta, el sistema no debe volver a mostrar la oferta durante la sesión activa.
