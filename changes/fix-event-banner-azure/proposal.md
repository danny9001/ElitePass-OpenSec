# Propuesta: Corrección de Visualización de Banners de Eventos en Azure Storage

## 1. Contexto Comercial / Reporte de Bug
Los banners de eventos subidos no se están mostrando en la plataforma. Se espera que estas imágenes residan en Azure Storage. Sin embargo, en el catálogo y carrusel de eventos, las peticiones HTTP resultan en errores 404 buscando los archivos en rutas locales como `/uploads/event-banners/...` en lugar de cargarlas de forma directa desde la URL absoluta de Azure Storage.

## 2. Impacto
- Los usuarios finales ven imágenes rotas o banners faltantes en el carrusel de eventos destacados y las tarjetas de eventos.
- Deterioro de la estética visual del sitio web público y del panel administrativo.

## 3. Comportamiento Esperado
- Cuando un banner de evento o afiche se sube, debe guardarse en Azure Storage (si está configurado).
- Al renderizarse en el frontend, se debe resolver la URL correcta (URL absoluta de Azure Storage en lugar de ruta local relativa de uploads) y cargarse a través del componente optimizado de Next.js (`next/image`), el cual debe permitir el hostname correspondiente en `images.remotePatterns`.
