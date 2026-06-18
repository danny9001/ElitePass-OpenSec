# Proposal: Sincronización automática de usuarios y empresas desde apps satélite hacia Identity

## Business Intent
Cuando una app satélite (POS, Reservas) crea un usuario staff o una empresa nueva, esa entidad debe existir automáticamente en `elitepass-identity` para poder autenticarse vía SSO. Sin este sync, los usuarios creados en apps satélite no podían iniciar sesión en el ecosistema.

## Scope
1. **elitepass-identity:** nuevo endpoint `POST /api/v1/sync` que acepta dos acciones:
   - `sync-user-empresa`: vincula un usuario a una empresa + genera licencia de app si no existe
   - `sync-empresa`: garantiza que la empresa y su licencia de app existan en identity
2. **elitepass-pos:** cliente HTTP `identity-sync.ts` + llamadas fire-and-forget en `usuarios.router` y `empresas.router`
3. **elitepass-reservas:** cliente `identity-client.ts` + llamada en `user-actions.ts` al crear usuario staff
4. **elitepass-payments:** agregado middleware `compression` (mejora de performance — cambio de oportunidad)
5. **elitepass-pos landing:** agregada app `ElitePass Mundial` al ecosistema + fix pantalla negra (dist desactualizado)
