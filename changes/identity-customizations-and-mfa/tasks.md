# Execution Tasks: Identity Customizations, Subscriptions, and Inactivity Session Timeout

## Phase 1: Environment & Dependency Setup
- [ ] Copy `AZURE_STORAGE_CONNECTION_STRING` and `AZURE_STORAGE_CONTAINER_NAME` from `/home/soporte/elitepass-reservas/.env` to `/home/soporte/elitepass-identity/.env`.
- [ ] Install `@azure/storage-blob` and `sharp` dependencies in `elitepass-identity`.
- [ ] Install `bcrypt` and `@types/bcrypt` (for password hashing/updating).

## Phase 2: Database Schema & Migration
- [ ] Add address and branding fields to the `Empresa` model in `prisma/schema.prisma`.
- [ ] Add `periodo` field to the `LicenciaSKU` model.
- [ ] Add `loginBgUrl`, `loginFooterText`, `loginTheme`, and `loginObjectsOrder` fields to the `App` model.
- [ ] Run `pnpm run db:push` to apply changes.
- [ ] Run a one-time migration script (or SQL query) to update all users whose `usageLocation` is null to `"BO"` (Bolivia).

## Phase 3: Image Upload Utility (Azure & Sharp WebP)
- [ ] Implement `src/lib/azure-upload.ts` with Sharp WebP maximum compression (lossless or 100% quality) and upload functionality.

## Phase 4: User (Persona) Form Refactor
- [ ] Hide/remove UPN field from forms.
- [ ] Set Bolivia ("BO") as the default location in forms.
- [ ] Integrate WebP image upload component for profile photos.
- [ ] Add multi-company selection control in user editing.

## Phase 5: Company (Empresa) Profile & Membership Refactor
- [ ] Add NIT, color palette, address fields, city, country, phone, and logo upload to the Empresa Form.
- [ ] Implement membership editor to select/remove people belonging to the company.
- [ ] Allow assigning licenses directly to companies in this section.

## Phase 6: Product & SKU Management Refinement
- [ ] Expose associated application, validity period, and status in the SKU manager.

## Phase 7: Contract Subscriptions Management
- [ ] Implement Subscription creation (immutable after creation).
- [ ] Implement soft-delete (setting status to `"eliminada"`).
- [x] Add a history tab/section showing active vs. deleted subscriptions.

## Phase 8: Login Customization & Brand Preview
- [x] Update the App branding card with fields for background image, footer text, theme, and layout object ordering.
- [x] Implement a live/interactive Branding Preview in the apps dashboard.
- [x] Apply the custom branding settings in the `/login` page (handling custom background, theme, footer, and order of fields).

## Phase 9: Super Admin Management Dashboard
- [x] Create a dedicated super-admin management view `/dashboard/super-admins`.
- [x] Implement admin/super-admin creation, editing (including WebP profile photo), and password resetting with bcrypt-12.

## Phase 10: Inactivity Session Timeout & MFA
- [x] Create client-side `InactivityTimeoutProvider` (5-minute inactivity tracker).
- [x] Implement server-side activity cookie validation.
- [x] Propose & implement MFA recommendations in the dashboard profile section.

## Phase 11: Telegram Alerts & "Sin Acceso" Page
- [x] Create dynamic Telegram alert utility `src/lib/telegram.ts`.
- [x] Intercept failed sign-in attempts in `src/app/api/[...all]/route.ts` and send Telegram alerts.
- [x] Redesign `/sin-acceso/page.tsx` with zinc theme styling and clean message.
- [x] Trigger Telegram alerts on unauthorized admin dashboard access via layout or `/sin-acceso` route.
- [ ] Verify clean compilation and build with `pnpm run build`.


