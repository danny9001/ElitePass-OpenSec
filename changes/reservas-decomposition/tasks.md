# Plan de Tareas: Migración Modular del Sistema de Reservas

Este plan detalla los pasos requeridos para llevar a cabo la descomposición modular del sistema de reservas de manera segura e incremental (Strangler Fig Pattern).

---

## Fase 1: Infraestructura de Ruteo y Aislamiento de Código (Backend)
- [ ] Configurar el ruteo unificado en Nginx para distribuir el tráfico de `/` (clientes), `/admin` (staff) y `/scanner` (puerta).
- [ ] Implementar la normalización de nombres de subida para vouchers y la generación de URLs firmadas temporales (SAS Tokens) para el contenedor privado de Azure Storage.
- [ ] Configurar el Event Broker local utilizando Redis Streams para despachar notificaciones y puntos de fidelidad en segundo plano de manera asíncrona.
- [ ] Crear la base del modelo de excepciones horarias por sectores y autorizaciones manuales en el modelo [Event](file:///home/soporte/elitepass-reservas/prisma/schema.prisma#L425).

## Fase 2: Desarrollo y Despliegue de la Interfaz de Clientes (Frontend)
- [ ] Desarrollar/Extraer el Portal de Clientes y los formularios de invitación como una PWA independiente en Vite/React (Puerto 3005).
- [ ] Integrar el límite de cupos dinámicos en los formularios de invitación (200 para Managers, y cupos asignados por evento para Relacionadores).
- [ ] Conectar los formularios públicos con la API del Monolito para que las solicitudes entren en la cola de aprobación en estado `PENDIENTE`.

## Fase 3: Optimización del Core de Reservas y Consumos (Panel Admin)
- [ ] Refactorizar el módulo de asignación de mesas para asegurar WebSockets en tiempo real y bloqueo de seguridad concurrente de 10 minutos ([ReservationLock](file:///home/soporte/elitepass-reservas/prisma/schema.prisma#L593)).
- [ ] Desarrollar la interfaz simplificada para cajeros dentro del panel que permita cargar manualmente las compras/consumos totales asociados a una mesa.
- [ ] Habilitar el panel de revisión visual de vouchers con firma temporal en el módulo de solicitudes.

## Fase 4: Despliegue de la PWA del Escáner y Validaciones
- [ ] Desarrollar el Escáner de Portería como SPA ligera (Puerto 3006) optimizada para cámaras móviles.
- [ ] Implementar la validación horaria del QR confrontando la hora del evento y aplicando las excepciones del sector (ej. sector Camel).
- [ ] Integrar la carga asíncrona y almacenamiento local (`localStorage`) de la lista de QRs aprobados de la noche para permitir escaneos veloces y tolerar pérdidas temporales de conexión a internet.
