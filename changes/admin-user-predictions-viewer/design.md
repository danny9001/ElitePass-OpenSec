# Design: Visor de Pronósticos por Usuario

## API
- **Ruta**: `GET /api/admin/user-predictions?userId=<id>`
- **Auth**: `getSessionUser()` → solo `superadmin`
- **Query**: JOIN predictions + matches, ordenado por `m.fecha ASC`
- **Campos devueltos**: id, pred_local, pred_visitante, puntos, match_id, local, visitante, fecha, estado, fase, grupo, goles_local, goles_visitante

## UI — `/src/app/admin/predictions/page.tsx`
- `use client` con `useRouter` para botón volver
- Carga lista de usuarios desde `/api/admin/users` (solo superadmin puede)
- Al seleccionar usuario → llama `GET /api/admin/user-predictions?userId=X`
- Tabla responsive: Partido | Pronóstico | Fase | Fecha | Puntos
- Puntos: badge verde (3pts exacto), amarillo (1pt resultado), rojo (0 pts), gris (pendiente)
- Enlace desde la pestaña Admin en page.tsx → botón "Ver Pronósticos" que navega a `/admin/predictions`
