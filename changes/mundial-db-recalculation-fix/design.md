# Design: World Cup Leaderboard Recalculation Fix

## Technical Diagnosis
- The frontend leaderboard queries the database using `SELECT ... WHERE u.activo = true AND u.participa = true`.
- The `recalculate_leaderboard()` PL/pgSQL function runs:
  ```sql
  WITH user_scores AS (
    SELECT u.id AS u_id, ...
    FROM users u
    LEFT JOIN predictions p ON u.id = p.user_id
    GROUP BY u.id
  ),
  ranked AS (
    SELECT u_id, ..., ROW_NUMBER() OVER (ORDER BY total_pts DESC, exact_cnt DESC, u_id ASC) as rk
    FROM user_scores
  )
  ```
- Because it lacks the filters, users who are inactive or toggled to "Viewer" (`participa = false`) are assigned ranks (e.g., rank 1). Since they are filtered out at the API level, the frontend displays ranks with gaps (e.g., rank 2, 3, but no rank 1).

## Implementation Details
1. **Clean up Leaderboard:** At the start of `recalculate_leaderboard()`, delete any entries for users who should not be ranked:
   ```sql
   DELETE FROM leaderboard
   WHERE user_id NOT IN (
     SELECT id FROM users 
     WHERE activo = true 
       AND participa = true
       AND (tipo != 'superadmin' OR EXISTS (SELECT 1 FROM user_companies WHERE user_id = id))
   );
   ```
2. **Filter users during ranking:** Apply the same conditions in `user_scores` subquery:
   ```sql
   FROM users u
   LEFT JOIN predictions p ON u.id = p.user_id
   WHERE u.activo = true
     AND u.participa = true
     AND (u.tipo != 'superadmin' OR EXISTS (SELECT 1 FROM user_companies WHERE user_id = u.id))
   ```
3. **Synchronization:** Apply these changes to the files `init.sql` and `generate_init_sql.js` on VM02-MUNDIAL and execute the function definition in the live database container `postgres`.
