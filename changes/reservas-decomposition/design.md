# Diseño Técnico: Arquitectura de Micro-aplicaciones con Dominio Único

Este documento describe la arquitectura detallada para estructurar y subdividir la plataforma [elitepass-reservas](file:///home/soporte/elitepass-reservas) en módulos desacoplados corriendo en un mismo servidor.

---

## 1. Arquitectura de Despliegue y Ruteo (Nginx Gateway)

Para mantener un único portal de cara al usuario, utilizaremos Nginx como proxy inverso. Nginx interceptará las solicitudes bajo el dominio único y las distribuirá a los microservicios locales de la siguiente forma:

```
                  ┌───────────────────────────────┐
                  │      Cliente / Navegador      │
                  └───────────────┬───────────────┘
                                  │ (HTTP / HTTPS)
                                  ▼
                  ┌───────────────────────────────┐
                  │         Nginx Proxy           │
                  │       (Puerto 80/443)         │
                  └───────────────┬───────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         │ /                      │ /admin                 │ /scanner
         ▼                        ▼                        ▼
┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│ Micro-app Client │    │  Panel Admin     │    │  Escáner PWA     │
│ (React - PWA)    │    │  (Next.js Core)  │    │  (SPA Ligera)    │
│ Puerto: 3005     │    │  Puerto: 3000    │    │  Puerto: 3006    │
└──────────────────┘    └──────────────────┘    └──────────────────┘
```

---

## 2. Estrategia de Persistencia (PostgreSQL & Prisma)

Mantendremos una única base de datos PostgreSQL física, pero implementaremos aislamiento lógico mediante esquemas organizados en Prisma:

* **Esquema Core / Eventos:** Tablas `Event`, `Sector`, `Table`, `Package`.
* **Esquema Reservas (Transaccional):** Tablas `Request`, `ReservationLock`, `Payment` (Vouchers).
* **Esquema Clientes & Fidelidad:** Tablas `Guest`, `LoyaltyPointLedger`, `LoyaltyReward`, `LoyaltyRedemption`.
* **Esquema Control de Accesos:** Tablas `QREntry`, `QRScan`.

### Control de Concurrencia de Mesas
Para la asignación manual de mesas, utilizaremos el modelo [ReservationLock](file:///home/soporte/elitepass-reservas/prisma/schema.prisma#L593) con un bloqueo optimista. Al iniciar el proceso de reserva en el frontend, se registra un bloqueo con una expiración de 10 minutos. Una restricción única compuesta `eventId` + `tableId` en la base de datos impedirá la inserción de registros duplicados en estado activo.

---

## 3. Almacenamiento Seguro de Vouchers (Azure Blob Storage)

Los vouchers de pago subidos por los clientes se guardarán en un contenedor privado de Azure Blob Storage.
* **Normalización de Nombres:** El servicio renombrará cada archivo en el momento de la subida para evitar colisiones y mantener el orden de auditoría:
  `voucher_{organizationId}_{requestId}_{timestamp}.{extension}`
* **Acceso Seguro (Visualización):** El administrador de reservas visualizará los comprobantes en el panel administrativo mediante la generación bajo demanda de un **Token SAS (Shared Access Signature)** con una validez temporal de 5 minutos. Las URLs directas nunca serán públicas.

---

## 4. Comunicación Asíncrona (Redis Streams)

Para acelerar la respuesta del servidor en operaciones complejas de gestión, utilizaremos la instancia local de **Redis 7** como Event Broker.

### Evento: `RESERVATION_APPROVED`
Cuando un Manager aprueba una reserva tras validar el voucher:
1. El backend de administración escribe en PostgreSQL y publica en Redis:
   `XADD reservation_stream * event RESERVATION_APPROVED payload <json_data>`
2. **Consumidores en segundo plano:**
   * **Servicio de Notificaciones:** Lee el evento de Redis y envía un mensaje de texto simple vía API al bot de Telegram.
   * **Servicio de Fidelidad:** Lee el evento, verifica los invitados confirmados y las compras de la mesa registradas manualmente por los cajeros, y calcula/asigna los puntos a la tabla [LoyaltyPointLedger](file:///home/soporte/elitepass-reservas/prisma/schema.prisma#L926).

---

## 5. Control de Excepciones de Acceso (QR)

El servicio de verificación de QR en puerta contrastará la hora del servidor con el límite `maxEntryTime` configurado en el evento.
* **Regla de Sector VIP (Excepciones):** Si el sector del pase escaneado está configurado como exento (ej. el sector *Camel*), el sistema aprobará el acceso automáticamente sin importar la hora de ingreso.
* **Aprobación de Extras:** El panel del escáner permitirá que un administrador con privilegios firme de forma digital una excepción temporal para autorizar ingresos tardíos directamente en portería.
