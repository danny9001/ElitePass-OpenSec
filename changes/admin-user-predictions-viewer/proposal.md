# Proposal: Visor de Pronósticos por Usuario (Admin)

## Business Intent
El superadmin necesita poder seleccionar cualquier usuario registrado y ver todos sus pronósticos con el detalle de: partido, pronóstico ingresado, fase y fecha. Esto permite auditoría, soporte y control de la actividad de los participantes.

## Scope
1. Nueva página dedicada `/admin/predictions` (page separada para no engordar page.tsx).
2. Solo accesible para `superadmin`.
3. Selector de usuario (dropdown con todos los usuarios del sistema).
4. Tabla con columnas: Partido · Pronóstico · Fase · Fecha · Puntos.
5. Nueva API route `/api/admin/user-predictions` que devuelve los pronósticos de un usuario específico.
