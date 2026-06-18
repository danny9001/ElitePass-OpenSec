# Diseño Técnico: Corrección de Visualización de Banners de Eventos en Azure Storage

## 1. Análisis de Causa Raíz
Hemos investigado los siguientes componentes de la arquitectura:
- **Base de Datos (PostgreSQL):** Todos los 5 eventos existentes tienen `bannerImage: null` en la base de datos.
- **Azure Storage (Contenedor `jet00`):** No existe el directorio/prefijo `event-banners` en la cuenta de almacenamiento (0 blobs). Otros directorios como `ticket-art`, `table-maps` y `vouchers` sí contienen archivos.
- **Acción del Servidor (`uploadEventBanner`):** Utiliza `uploadBlob` para subir a Azure con el prefijo `event-banners/`.
- **Formulario de Evento (`event-form-page.tsx`):**
  - El esquema de validación (`eventFormSchema`) define `bannerImage` como `z.custom<File>().optional()`.
  - En el método `onSubmit`, si existe `values.bannerImage` (que es de tipo `File`), se sube a Azure Storage y se obtiene la URL absoluta, la cual se guarda en base de datos.
  - Sin embargo, en el método `form.reset(...)` (utilizado al cargar/inicializar el formulario en modo edición), no se setean los valores del formulario para las propiedades del tipo `File` (`image`, `bannerImage`, `paymentQR`, `ticketArt`, `tableMap`). Aunque las previsualizaciones visuales se configuran correctamente mediante estados independientes (ej. `bannerImagePreview`), el valor del formulario se mantiene como `undefined`.

## 2. Decisión de Diseño
Para resolver el problema y garantizar que los banners se suban a Azure Storage y se guarden/muestren de forma robusta, realizaremos las siguientes acciones:
1. **Comprobar la creación/actualización con banner:** Validar mediante un script de prueba si la creación y actualización de eventos con banner funciona correctamente a nivel de base de datos y Azure Storage.
2. **Corrección de Inicialización en Edición:** Asegurar que cuando el formulario de edición se inicialice, se mantenga constancia del valor actual del banner (como string o indicador) para evitar cualquier blanqueo o inconsistencia, o simplemente dejar que se comporte de forma transparente.
3. **Optimización en Frontend:** Verificar que el componente de carrusel y de tarjetas públicas renderice el banner correctamente cuando esté presente, utilizando el hostname de Azure Storage configurado en `next.config.js`.
4. **Verificación de hostname de Next.js:** Asegurar que PM2 cargue las variables de entorno de Azure Storage para que `images.remotePatterns` se configure con el hostname dinámico de Azure en producción.
