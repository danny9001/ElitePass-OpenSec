# Design: Deferred Score Notifications

## Decisión Técnica

### Archivo principal modificado
`src/lib/sync.ts`

### Tipo acumulador

```typescript
type PendingGoalNotif = {
  matchId: number;
  local: string;
  visitante: string;
  golesLocal: number;
  golesVisitante: number;
  baselineGolesLocal: number;   // score PRE-sync para identificar quién marcó
  baselineGolesVisitante: number;
  isFinished: boolean;
};
```

### Flujo modificado

**Antes:**
```
Fuente A detecta gol → escribe DB → envía push [SCORE A]
Fuente B detecta gol → escribe DB → envía push [SCORE B]
Usuario recibe 2 pushes (posiblemente con scores incorrectos)
```

**Después:**
```
Fuente A detecta gol → escribe DB → acumula en Map[matchId]
Fuente B detecta gol → escribe DB → sobrescribe Map[matchId] con score final
flushPendingNotifications() → envía UNA push con score final confirmado
```

### Separación SSE vs Push

- `broadcastUpdate('goal', ...)` **NO se difiere** → UI en tiempo real sigue actualizándose por fuente
- `sendPushToAllActive(...)` **SÍ se difiere** → una sola notificación al final

### Fallback (backward compatibility)

Las 4 funciones fuente aceptan `pendingGoalNotifs?: Map<...>`. Si se llaman sin el map (e.g., debug o test directo), caen al comportamiento original de notificación inmediata.

### Tabla de auditoría

```sql
CREATE TABLE IF NOT EXISTS score_change_log (
  id SERIAL PRIMARY KEY,
  match_id INTEGER REFERENCES matches(id) ON DELETE CASCADE,
  source VARCHAR(50) NOT NULL,
  old_goles_local INTEGER,
  old_goles_visitante INTEGER,
  new_goles_local INTEGER NOT NULL,
  new_goles_visitante INTEGER NOT NULL,
  estado VARCHAR(20) NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Creada via `CREATE TABLE IF NOT EXISTS` dentro de `syncMatches()` al inicio.

### Sistema de logs

Categoría nueva `'SYNC'` en `system_logs`:
- Detectado por fuente: `[ESPN] Gol detectado: ...`
- Push enviado: `Push enviado (gol): ...`

Consulta forense: `SELECT * FROM system_logs WHERE categoria = 'SYNC' ORDER BY created_at DESC`
