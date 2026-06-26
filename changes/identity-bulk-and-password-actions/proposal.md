# Proposal: User Password Management and Bulk Actions in Identity

## 1. Business Context & Problem Statement
Currently, administrators of the ElitePass platform do not have an option to reset or change the password of registered users (Personas) within the Identity provider dashboard (`elitepass-identity`). Additionally, admins cannot perform administrative actions in bulk, such as setting a password for multiple users at once or executing bulk deletions. This lack of features creates operational bottlenecks when onboarding groups of staff or resetting passwords for credentials-based users.

## 2. Proposed Solution
1. **Individual Password Management**:
   - Add a "Cambiar contraseña" (Change password) action in user detail page (`/dashboard/personas/[id]`) and user row action menu (`PersonasTable`).
   - The password change form will enforce security rules (minimum 8 characters, hashed using `bcrypt` with salt round 12) and record an audit log event (`user_password_changed`).

2. **Bulk Actions on Users**:
   - Add selection checkboxes to each user row in the desktop table in the User List (`/dashboard/personas`).
   - Implement a floating or top bulk action bar that appears when one or more users are selected.
   - Support the following bulk actions:
     - **Bulk Change Password**: Set a new password for all selected users.
     - **Bulk Delete**: Delete all selected users (with proper admin warnings and cascading verification).
   - Each action will check admin authorization via `requireAdmin()`, perform updates in safe Prisma transactions, and register audit log entries.

## 3. Scope of Changes
- **Backend Actions (`src/lib/actions.ts`)**:
  - Add `updatePersonaPassword(userId, password)` server action.
  - Add `bulkUpdatePersonaPasswords(userIds, password)` server action.
  - Add `bulkDeletePersonas(userIds)` server action.
- **Frontend Pages/Components**:
  - `src/components/PersonasClient.tsx`: Add row selection checkboxes, bulk action bar, and a bulk password edit modal/dialog. Add password change action to individual user actions.
  - `src/app/dashboard/personas/[id]/page.tsx`: Add a button to change the individual's password directly from the user detail view.
