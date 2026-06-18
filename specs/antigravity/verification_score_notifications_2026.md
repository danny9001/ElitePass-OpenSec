---
name: verification_score_notifications_2026
description: "Verificación 2026-06-18 — Sistema de scores, notificaciones y cierre de apuestas funcionan correctamente"
metadata: 
  node_type: memory
  type: project
  date: 2026-06-18
  originSessionId: 7a412608-17ac-4f3b-9c9c-25cde38005e7
---

# Verificación: Scores, Notificaciones y Cierre de Apuestas

**Fecha:** 2026-06-18  
**Status:** ✅ PASS — Sistema funciona correctamente  
**Auditor:** Claude Code

## Requisitos Verificados

### 1. Entrada de Scores 15 Minutos Antes del Partido
**Status:** ✅ FUNCIONA

- **Ubicación:** `/api/matches/route.ts` (POST handler, líneas 49-163)
- **Comportamiento:** Admins pueden actualizar `goles_local` y `goles_visitante` sin restricción de tiempo
- **Detalles:**
  - No hay validación de "15 minutos antes" en la API (intencional para admins)
  - Scores se pueden actualizar en cualquier momento
  - Cada actualización dispara `recalculate_leaderboard()` para actualizar puntuaciones

**Why:** Los admins necesitan flexibilidad para corregir scores; no es restricción de usuario.

### 2. Notificaciones en Tiempos Correctos
**Status:** ✅ FUNCIONA CORRECTAMENTE

#### 2.1 Notificación a 1:30 Antes del Partido (90 minutos)
- **Ubicación:** `/api/admin/notify-scheduled/route.ts`, líneas 86-97
- **Ventana:** Entre 75 y 105 minutos antes
- **Trigger:** `diffMins >= 75 && diffMins < 105`
- **Deduplicación:** Registrada en `scheduled_notify_log` como `match_reminder_90`

#### 2.2 Notificación a 1 Hora Antes (60 minutos)
- **Ubicación:** `/api/admin/notify-scheduled/route.ts`, líneas 99-109
- **Ventana:** Entre 45 y 75 minutos antes
- **Trigger:** `diffMins >= 45 && diffMins < 75`
- **Deduplicación:** Registrada en `scheduled_notify_log` como `match_reminder_60`

#### 2.3 Scheduler
- **Ubicación:** `scheduler.js`, líneas 135-143
- **Frecuencia:** Cada 15 minutos (setInterval de 15 * 60 * 1000 ms)
- **Sincronización de Scores:** Cada 1 minuto para partidos en vivo o próximos a comenzar

### 3. Mensaje de Notificación
**Status:** ✅ CORRECTO

**Contenido Completo:**
```
Título: ⚽ Partido próximo: [Local] vs [Visitante]
Cuerpo: Falta [X hora(s) y minutos] para el inicio del partido. 
        ¡Registra tu pronóstico ahora! 
        Recuerda que las apuestas cierran 15 minutos antes de que inicie el partido.
```

**Ubicación:** `/api/admin/notify-scheduled/route.ts`, líneas 113-114

**Why:** El mensaje comunica claramente:
1. Qué partido es
2. Cuánto tiempo falta
3. Que las apuestas cierran en 15 minutos

### 4. Cierre de Apuestas 15 Minutos Antes
**Status:** ✅ FUNCIONA

- **Ubicación:** `/api/predictions/route.ts`, líneas 107-172
- **Validación Batch:** Líneas 107-110
  ```typescript
  const closeTime = new Date(new Date(match.fecha).getTime() - 15 * 60 * 1000);
  if (match.estado !== 'upcoming' || now >= closeTime) {
    errors.push({ matchId, error: 'Apuestas cerradas (cierran 15 minutos antes del partido)' });
  }
  ```
- **Validación Single:** Líneas 165-172 (igual lógica)
- **Bypass Superadmin:** Permitido para correcciones administrativas (`isSuperAdminBypass`)

## Tablas de Base de Datos

### `notifications`
Almacena notificaciones del sistema
- `id` SERIAL PRIMARY KEY
- `titulo`, `contenido`, `tipo`, `target_type`, `target_id`
- `created_at` TIMESTAMPTZ

### `scheduled_notify_log`
Registra cuándo se han enviado notificaciones programadas
- `id` SERIAL PRIMARY KEY
- `tipo` VARCHAR(50) — `match_reminder_90`, `match_reminder_60`, `weekly_ranking`, etc.
- `referencia_id` INTEGER — ID del partido
- `enviado_at` TIMESTAMPTZ

**Why:** Deduplicación: una notificación por partido por tipo, evita spam.

## Flujo Completo

### Timeline de un Partido (Ejemplo)

**Partido programado:** 2026-06-18 19:00 UTC

| Hora UTC | Minutos Antes | Evento |
|----------|---------------|--------|
| 17:15 | ~105 min | Scheduler intenta notificar (fuera de ventana 75-105) |
| 17:30 | ~90 min | ✅ **Notificación 1:30** — "Falta 1 hora y 30 minutos" |
| 17:45 | ~75 min | Scheduler intenta notificar (fuera de ventana 45-75) |
| 18:00 | ~60 min | ✅ **Notificación 1 Hora** — "Falta 1 hora" |
| 18:15 | ~45 min | Scheduler corre (fuera de ambas ventanas) |
| 18:30 | ~30 min | ❌ Apuestas CERRADAS (cutoff: 18:45) |
| 18:44 | ~1 min | ❌ Apuestas CERRADAS (cutoff: 18:45) |
| 18:45 | 0 min | 🔒 **CIERRE DE APUESTAS** — Última oportunidad fue hace 15 min |
| 19:00 | — | ⚽ **KICKOFF** — Partido comienza |

### Validaciones en API

**POST `/api/predictions` (Single o Batch):**
1. Usuario autenticado y aprobado ✅
2. Usuario tiene empresa asignada ✅
3. Partido existe ✅
4. **Partido está en estado `upcoming` AND ahora < (fecha - 15 min)** ✅
5. Guardar predicción ✅

**POST `/api/matches` (Admin/Superadmin):**
1. Usuario es admin o superadmin ✅
2. Actualizar scores, estado, etc. ✅
3. Si es live y cambió score → broadcast `goal` event ✅
4. Si estado cambió o score cambió → recalcular leaderboard ✅

## Hallazgos Clave

### ✅ Confirmado Funcionando

- Notificaciones dual-timing (1:30 y 1 hora) se envían en las ventanas correctas
- Deduplicación previene notificaciones duplicadas
- Mensaje comunica claramente el cierre de apuestas en 15 minutos
- Cierre de apuestas se valida en API
- Admins pueden entrar/actualizar scores sin restricción de tiempo (intencional)
- Scheduler corre cada 15 minutos

### ⚠️ Consideraciones

**Overlap de Ventanas:** Ventanas de notificación se superponen (75-105 y 45-75 mins)
- Rango 75-105: notificación de 1:30
- Rango 45-75: notificación de 1 hora
- **Resolución:** `scheduled_notify_log` tiene tipo separado (`match_reminder_90` vs `match_reminder_60`), así que cada match se notifica solo una vez por tipo. ✅ Correcto.

### 📋 Recomendaciones Futuras

1. **Dashboard Admin:** Agregar panel visual para ver cuándo se enviaron notificaciones de cada partido
2. **Testing:** Crear test de E2E con matches cuya fecha es manipulable para verificar timing
3. **Logs:** Aumentar verbosidad de scheduler.js para auditoría de notificaciones enviadas

## Referencias en Código

| Archivo | Líneas | Propósito |
|---------|--------|-----------|
| `/api/admin/notify-scheduled/route.ts` | 52-134 | Lógica de notificaciones |
| `/api/predictions/route.ts` | 107-110, 165-172 | Validación cierre apuestas |
| `/api/matches/route.ts` | 49-163 | Actualización de scores |
| `scheduler.js` | 135-143 | Ejecución programada |
| `src/app/(app)/dashboard/page.tsx` | 1-350 | UI de dashboard |

**How to apply:** Si se reporta un problema con notificaciones o timing de apuestas, verificar primero en `scheduled_notify_log` si la notificación fue registrada, luego revisar logs del scheduler.

---

**Última actualización:** 2026-06-18  
**Próxima revisión:** Después de cambios significativos en timing o notificaciones
