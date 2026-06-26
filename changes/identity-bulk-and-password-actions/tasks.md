# Tasks: User Password Management and Bulk Actions in Identity

- [x] Implement backend server actions in `/home/soporte/elitepass-identity/src/lib/actions.ts` <!-- id: 1 -->
  - [x] Add `updatePersonaPassword(userId, password)`
  - [x] Add `bulkUpdatePersonaPasswords(userIds, password)`
  - [x] Add `bulkDeletePersonas(userIds)`
- [x] Update frontend UI components in `/home/soporte/elitepass-identity/src/components/PersonasClient.tsx` <!-- id: 2 -->
  - [x] Create `ChangePasswordModal` component for password entry.
  - [x] Add password change icon/button trigger to `PersonaActions` (affects table rows & mobile cards).
  - [x] Add bulk actions checkboxes and logic to `PersonasList` and `PersonasTable`.
  - [x] Create floating/top `BulkActionToolbar` component to show when items are selected.
  - [x] Implement bulk change password action modal.
  - [x] Implement bulk delete confirmation modal.
- [x] Add password reset/change button to user detail page in `/home/soporte/elitepass-identity/src/app/dashboard/personas/[id]/page.tsx` <!-- id: 3 -->
- [x] Build & restart application <!-- id: 4 -->
  - [x] Run `pnpm build` inside `elitepass-identity` to check typescript / compilation correctness.
  - [x] Run `pm2 reload elitepass-identity` to apply server-side updates.
- [x] Verify functionality and log memory <!-- id: 5 -->
  - [x] Check console and build outputs for warnings/errors.
  - [x] Log status to `MEMORY.md`.

## Execution Details
- **Backend actions**: Created `updatePersonaPassword`, `bulkUpdatePersonaPasswords`, and `bulkDeletePersonas` in `/home/soporte/elitepass-identity/src/lib/actions.ts` utilizing `requireAdmin()` access restriction and `logAudit()` integration.
- **Frontend list page**: Integrated selection checkbox states into `PersonasList`, `PersonasTable`, and `PersonaCard` inside `PersonasClient.tsx`. Created an interactive and beautiful floating bulk action toolbar matching the ElitePass design system.
- **Frontend detail page**: Imported and placed `ChangePasswordButton` component in the header of `/home/soporte/elitepass-identity/src/app/dashboard/personas/[id]/page.tsx`.
- **Validation**: Project compiled successfully via `pnpm build` and the PM2 cluster was reloaded. Verified database records for user passwords and SSO login capability.
