# Proposal: Fix Stale Score Push Notifications

## Fecha: 2026-06-24

## Problema

En el partido de Portugal del 2026-06-24, el usuario recibió una notificación push diciendo "2-1" cuando el marcador real era 3-0, y tuvo que corregir manualmente el score en la base de datos.

## Causa Raíz

El sistema tiene 4 fuentes externas (ESPN, 365Scores, FixtureDownload, FootballData) que se ejecutan **secuencialmente** en `syncMatches()`. Cada fuente detecta cambios de score de forma independiente y envía su propia notificación push inmediatamente.

Los keys de deduplicación en `match_notif_log` son basados en el score específico (`goal_2_1`, `goal_3_0`) — son **distintos**, por lo tanto ambas notificaciones pasan el filtro.

Secuencia exacta del bug:
1. ESPN (primera en ejecutar) recibe score 2-1 (dato retrasado de su API) → escribe 2-1 en DB → key `goal_2_1` → **push "2-1" enviado** ← BUG
2. 365Scores recibe score 3-0 → escribe 3-0 en DB → key `goal_3_0` → push "3-0" enviado
3. El usuario recibió la notificación falsa 2-1 primero

## Impacto

- Usuarios reciben información incorrecta vía push
- Push notifications no se pueden revocar una vez enviadas
- Requiere corrección manual del administrador
- Sin trail de auditoría para diagnosticar la causa

## Solución Propuesta

1. **Notificaciones diferidas**: Cada fuente acumula los cambios de score en un Map compartido (last-write-wins por partido). Al finalizar todas las fuentes, se envía UNA sola notificación con el score final confirmado.

2. **Logging de auditoría**: Agregar `logSystem()` con categoría 'SYNC' en cada detección de gol, para tener trail forense.

3. **Tabla `score_change_log`**: Registro persistente de cada cambio de score detectado por cada fuente, con before/after values.

**Nota importante**: Los updates en tiempo real (SSE via `broadcastUpdate('goal', ...)`) NO se difieren — el scoreboard en vivo de la aplicación sigue actualizándose inmediatamente con cada fuente. Solo se difieren las **push notifications** (que no pueden revocarse).
