# Tasks: Database Backup Fix

- [x] Implement BigInt serialization replacer in `src/lib/backups/database-backup.ts` <!-- id: 0 -->
- [x] Bypass CSRF checks for `/api/backups/` in `src/middleware.ts` <!-- id: 7 -->
- [x] Run diagnostic script to verify full and incremental backups succeed locally <!-- id: 1 -->
- [x] Run Biome formatting check and fix (`npx biome check --write`) <!-- id: 2 -->
- [x] Bump package.json version to `1.4.89` <!-- id: 3 -->
- [x] Compile the application (`pnpm build`) to verify there are no type errors <!-- id: 4 -->
- [ ] Restart the Next.js app on PM2 (`pm2 restart elitepass-reservas`) <!-- id: 5 -->
- [ ] Run manual API test curl to verify backups complete via API `/api/backups/run` <!-- id: 6 -->
