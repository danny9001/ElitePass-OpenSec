# Design: Compile, Commit, Push, and Deploy ElitePass Identity

## Deployment Steps
1. **Compilation Check:** Run `pnpm run build` on `elitepass-identity` to verify there are no compilation/TypeScript errors.
2. **Version Bump:** Increment the version in `package.json` from `1.0.0` to `1.0.1`.
3. **Commit & Push:** Stage all untracked and modified files, commit with `chore: bump version 1.0.1`, and push to `origin/main`.
4. **PM2 Deploy:** Restart `elitepass-identity` (PM2 ID `7`) to apply the changes.
