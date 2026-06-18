# Proposal: Identity User-Company Roles, App Display, and Global Light Theme

## Business Intent & Goals
1. **Application-Company Association Visibility**:
   When administrators are editing/assigning companies to a user ("Persona") in the Identity portal, they need to see exactly which applications each company is associated with. This prevents assigning a user to a company that doesn't have the required app access.
   
2. **Correct Directory Mapping for CLIENT and STAFF Users**:
   When users are linked to companies under Identity, their classification as either `CLIENT` (Cliente) or `STAFF` (Staff) must be correctly mapped to the database membership table (`PersonaEmpresa.tipo`). This ensures that satellite apps (e.g., Reservas or POS) can query and show them under their respective directories and authorize their access appropriately.
   
3. **Identity Theme Standard**:
   The Identity application must be forced to use a clean and consistent Light Theme (Tema Claro) across all pages.

## Scope
- Modify the `PersonaForm` to query and show associated applications for each company.
- Map the user account type (`CLIENT` to `"cliente"`, `STAFF` to `"staff"`) in `createPersona` and `updatePersona` inside `actions.ts`.
- Update `createEmpresa` and `updateEmpresa` in `actions.ts` to query the user account types and set the correct `tipo` role when adding personas to a company.
- Update `RolSelector` component in `elitepass-identity` to support the `"cliente"` role in its dropdown options.
- Update global CSS styles in `globals.css` to override tailwind dark mode selectors and variables to enforce a premium light theme.
- Ensure the application builds successfully, and then commit, push, and restart the service via PM2.
