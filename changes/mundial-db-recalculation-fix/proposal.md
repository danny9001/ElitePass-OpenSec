# Proposal: Fix Leaderboard Recalculation in apuestas-mundial-2026

## Business Intent
The World Cup application (`apuestas-mundial-2026`) recently introduced a toggle to mark users as "viewers" (`participa = false`), excluding them from the rankings. 
However, the database function `recalculate_leaderboard()` (defined in `init.sql` and installed in PostgreSQL) still computes ranks and positions for all users, including viewers and inactive accounts. When the API returns the filtered leaderboard, this mismatch results in gaps and incorrect ranking numbers.
This task aims to update the database function and initialization scripts to correctly exclude viewers and inactive users from the rank calculations and delete them from the leaderboard table.

## Scope of Changes
- Modify the PL/pgSQL function `recalculate_leaderboard()` to:
  - Delete entries from the `leaderboard` table for users who are no longer active, participating, or are superadmins without a company membership.
  - Filter the user selection using `WHERE u.activo = true AND u.participa = true AND (u.tipo != 'superadmin' OR EXISTS (SELECT 1 FROM user_companies WHERE user_id = u.id))` before computing ranking indices.
- Apply this fix to:
  - The live PostgreSQL database on VM02.
  - The [init.sql](file:///home/soporte/apuestas-mundial-2026/init.sql) file.
  - The [generate_init_sql.js](file:///home/soporte/apuestas-mundial-2026/generate_init_sql.js) file.
- Re-run the leaderboard recalculation on the database to fix current standings.
