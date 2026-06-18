# Propuesta: Descomposición Modular del Sistema de Reservas

## 1. Intención de Negocio
El objetivo es optimizar y estabilizar el rendimiento de la plataforma [elitepass-reservas](file:///home/soporte/elitepass-reservas) mediante la transición de un monolito Next.js clásico a una arquitectura modular de micro-aplicaciones. Esto garantizará que los flujos más dinámicos y recurrentes (como la **Gestión de Reservas** por parte de administradores y los formularios públicos de clientes) operen con alto rendimiento, sin verse afectados por picos de tráfico puntuales (como la validación de QRs en puerta durante eventos).

## 2. Reglas de Negocio Incorporadas

### A. Gestión de Reservas y Mesas
* **Asignación Manual:** La selección de mesas en el mapa de sectores se realiza de forma manual por el staff.
* **Control de Concurrencia:** Evitar reservas duplicadas de una misma mesa para un evento mediante bloqueos optimistas basados en la base de datos.
* **Validación de Pagos:** La verificación de pagos es manual. Los clientes cargan una imagen del voucher de pago. El staff del club descarga, revisa visualmente y cambia el estado de la reserva a `APROBADA`.
* **Registro de Consumo:** Los cajeros registran manualmente los consumos/compras totales de la mesa en el panel de reservas, lo cual genera puntos de fidelidad adicionales para el titular.

### B. Invitaciones con Formulario (Llenado Dinámico)
* **Jerarquía de Cupos:**
  * Managers y superiores: Límite máximo de 200 invitaciones por evento.
  * Team Leaders y Relacionadores: Límite de cupo configurable por evento.
* **Múltiples Formularios:** Los relacionadores pueden crear múltiples formularios de invitación personalizados como estrategia para incentivar la asistencia en noches de bajo flujo.
* **Flujo de Aprobación:** Todas las solicitudes recibidas a través de estos formularios entran a una cola de revisión y deben ser aprobadas manualmente por un Manager o superior.

### C. Fidelización e Invitados
* **Puntos Individuales:** Los puntos acumulados por la asistencia de invitados se acreditan directamente a la cuenta personal de cada invitado de manera individual cuando su QR es validado en la puerta.

### D. Control de Acceso (QR) y Excepciones
* **Restricción Horaria:** Se define una hora límite de ingreso al crear cada evento.
* **Excepciones de Entrada:** El administrador tiene la capacidad de autorizar ingresos tardíos (extra-horario) y definir excepciones automáticas basadas en sectores (ej. el sector *Camel* no tiene restricciones de horario).

## 3. Beneficios Esperados
* **Aislamiento de Carga:** El panel administrativo y el portal de clientes no sufrirán degradación de velocidad durante los periodos de validación intensiva de QR en puerta.
* **Navegación Unificada:** El usuario final percibirá un portal único bajo el mismo dominio principal, sin saber que está interactuando con servidores o micro-aplicaciones desacopladas por detrás.
* **Facilidad de Auditoría:** Contenedor privado de Azure Blob Storage estructurado bajo una normalización estricta de nombres para auditorías rápidas de vouchers de pago.
