# Tasks: World Cup Leaderboard Recalculation Fix

- [x] Edit `init.sql` on VM02 to update the `recalculate_leaderboard()` function definition.
- [x] Edit `generate_init_sql.js` on VM02 to update the `recalculate_leaderboard()` function definition in the SQL builder logic.
- [x] Apply the updated SQL function definition to the database inside the VM02 `postgres` container.
- [x] Execute `SELECT recalculate_leaderboard();` in the database to recalculate the positions based on active participating users.
- [x] Verify that the leaderboard has no positions assigned to viewers/inactive users.
- [x] Rebuild and deploy the VM02 Podman cluster app container and PM2 backup to ensure it works correctly.
