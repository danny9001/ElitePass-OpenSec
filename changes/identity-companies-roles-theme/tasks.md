# Tasks: Identity Companies App Display, Role Mapping, and Theme Customization

## Phase 1: Code Verification & Logic Adjustments
- [x] Verify that associated apps are queried in `getEmpresasActivas` and displayed next to company names in `PersonaForm.tsx`.
- [ ] Update `createPersona` and `updatePersona` in `src/lib/actions.ts` to map `accountType === "CLIENT"` to membership `tipo = "cliente"`.
- [ ] Update `createEmpresa` and `updateEmpresa` in `src/lib/actions.ts` to fetch user account types and set correct membership roles when adding users.
- [ ] Update `RolSelector.tsx` to include `cliente` in the dropdown options and remove normalization.

## Phase 2: Theme Verification
- [x] Ensure global light theme overrides are configured in `src/app/globals.css`.

## Phase 3: Build & Deployment
- [ ] Run `pnpm run build` in `/home/soporte/elitepass-identity` to verify compilation.
- [ ] Bump version to `1.0.2` in `package.json`.
- [ ] Stage, commit, and push modifications in `/home/soporte/elitepass-identity` to `origin main`.
- [ ] Restart `elitepass-identity` service via PM2 to deploy.
