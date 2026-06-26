# Proposal: Chat Global en Vivo + Stats Compactas Mobile

## Business Intent

1. **Chat Global** — Los usuarios quieren hablar entre sí durante el torneo, no solo por partido. Se necesita un chat general siempre visible (sin entrar a un partido) con indicador de quién está en línea, accesible desde cualquier pantalla como widget flotante.

2. **Stats Compactas Mobile** — En el inicio (dashboard), los 4 cards de estadísticas personales (Puntos, Posición, Predicciones, Aciertos) ocupan demasiado espacio en mobile. Se necesita una vista ultra-compacta en una sola fila para mobile, manteniendo los cards completos en desktop.

## Scope

### Feature 1: Chat Global
- Botón flotante (bottom-right) siempre visible para usuarios aprobados
- Modal full-screen al presionar el botón
- Lista de usuarios en línea (sidebar en desktop, collapsible en mobile)
- Chat en tiempo real con polling PostgreSQL (cluster-safe, sin Redis)
- Mensajes con avatar, nombre, hora
- Sin restricción por partido — es el chat general del torneo
- Solo usuarios aprobados pueden chatear

### Feature 2: Stats Compactas Mobile
- En mobile (`sm:hidden`): barra horizontal compacta de 4 métricas en una línea
- En desktop (`hidden sm:grid`): cards actuales sin cambios
- Métricas: Predicciones hechas · Puntos totales · Aciertos exactos · Posición #N
