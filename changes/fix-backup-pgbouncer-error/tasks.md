# Implementation Tasks: Fix Backup PgBouncer Error

- [x] Create/update specs in `/home/soporte/openspec/specs/backups/spec.md`
- [x] Implement `getCleanDatabaseUrl()` helper in `src/lib/backups/database-backup.ts`
- [x] Update `createFullBackupBuffer` in `src/lib/backups/database-backup.ts` to use `getCleanDatabaseUrl()`
- [x] Rebuild the application (`npm run build` or `pnpm build`) to compile changes
- [x] Test the backup execution from the UI or via route/script execution to verify it works without errors
- [x] Bump version in `package.json` following the official ecosytem versioning rule (PATCH +1)
- [x] Commit and push changes to GitHub
