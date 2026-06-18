# Technical Design: Identity Company Apps, Role Mapping, and Light Theme overrides

## 1. Application-Company Display in Persona Selection
- The query `getEmpresasActivas` in `actions.ts` is updated to include the active application associations (`apps { app { nombre } }`).
- In `PersonaForm.tsx`, the checkbox labels are modified to display the company name followed by the list of associated active applications in parentheses.

## 2. Directory Mapping for CLIENT and STAFF
- In `createPersona` and `updatePersona` (inside `actions.ts`), when assigning the user to companies, we check the `accountType` of the user. We set the `PersonaEmpresa` membership `tipo` to `"cliente"` if `accountType === "CLIENT"`, and to `"staff"` if `accountType === "STAFF"`. All other types fall back to `"miembro"`.
- In `createEmpresa` and `updateEmpresa` (inside `actions.ts`), when users are associated with a company, we query their `accountType` from the database and map the membership `tipo` similarly.
- In `src/components/RolSelector.tsx`, we add `{ value: "cliente", label: "Cliente" }` to the `ROLES_EMPRESA` array and remove the legacy normalization that redirected `"cliente"` to `"miembro"`.

## 3. Global Light Theme forcing
- In `src/app/globals.css`, we configure overrides that target the root elements and standard Tailwind v4 dark utility classes.
- Explicit overrides map `.bg-gray-950`, `.bg-zinc-950`, `.bg-gray-900`, `.bg-zinc-900`, `.bg-gray-800`, `.bg-zinc-800` to light-mode colors (e.g. `#f9fafb`, `#ffffff`, `#f3f4f6`).
- Dark text classes are mapped to dark gray values (`#111827`, `#1f2937`), and exceptions are declared so that buttons or links with high-contrast colored backgrounds (`.bg-indigo-600`, `.bg-blue-600`, etc.) preserve their white text.
