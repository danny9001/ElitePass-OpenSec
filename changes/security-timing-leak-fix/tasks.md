# Tasks: Security Timing Leak Fix

- [x] Modify `src/app/api/backups/run/route.ts` to use hash-based comparison <!-- id: 0 -->
- [x] Modify `src/app/api/cron/archive/route.ts` to use hash-based comparison <!-- id: 1 -->
- [x] Modify `src/app/api/audit-logs/archive/route.ts` to use hash-based comparison <!-- id: 2 -->
- [x] Modify `src/app/api/internal/rate-limit/route.ts` to use hash-based comparison <!-- id: 3 -->
- [ ] Verify that the Next.js production build succeeds with these changes <!-- id: 4 -->
- [ ] Restart the PM2 process (`pm2 restart elitepass-reservas`) <!-- id: 5 -->
- [ ] Run manual API test curl for backups, cron, and rate-limit endpoints using valid and invalid tokens <!-- id: 6 -->
