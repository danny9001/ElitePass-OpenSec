# Proposal: Refactor page.tsx Monolito → Páginas Separadas

## Business Intent
El archivo `src/app/page.tsx` tiene ~5 900 líneas en un único componente cliente. Esto causa:
- Bundle inicial pesado (todo el JS se descarga aunque el usuario solo vea el dashboard)
- Tiempos de compilación lentos
- Dificultad para mantener y escalar el código

El objetivo es separar cada tab en su propia ruta Next.js para que el code splitting funcione y cada módulo solo descargue lo que necesita.

## Decisión de diseño
- Mantener la **shell** en `page.tsx` con nav + auth check + layout global
- Cada tab se convierte en una página dedicada en `/dashboard`, `/partidos`, `/fixture`, `/ranking`, `/reglas`, `/perfil`, `/admin`
- El estado compartido crítico (`user`, `matches`, `predictions`, `leaderboard`) pasa a un **Context** liviano o cada página lo fetch por su cuenta
- La navegación pasa de `setActiveTab` a `router.push()`
- El SSE de realtime se extrae a un custom hook `useRealtime()` que cada página puede consumir
