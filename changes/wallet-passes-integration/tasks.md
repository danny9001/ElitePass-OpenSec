# Tasks: Wallet Passes Integration (Google & Apple Wallet)

## Phase 1: Microservice Setup & Certificates
- [ ] Create folder `/home/soporte/elitepass-billeteraQR` and initialize project.
- [ ] Install dependencies (`express`, `prisma`, `passkit-generator`, `jsonwebtoken`, `googleapis`, `dotenv`).
- [ ] Configure `tsconfig.json` and `.env` template.
- [ ] Securely request and place Apple Certificates in `/certs` (`pass.pem`, `privateKey.pem`, `wwdr.pem`).
- [ ] Securely request and place Google Service Account credentials JSON in `/certs/google-creds.json`.

## Phase 2: Cryptographic Builders
- [ ] Implement Apple Wallet `.pkpass` generator function (using `passkit-generator`).
- [ ] Map client details (VIP, role, name, companies) to Apple `pass.json` fields.
- [ ] Implement user avatar conversion and logo image rendering onto the Apple pass.
- [ ] Implement Google Wallet JWT signer function using the Google Cloud private key.
- [ ] Map the corresponding fields to Google Generic Pass Object layout.

## Phase 3: REST API Endpoints
- [ ] Create Express server on port `3300`.
- [ ] Implement API key protection for internal calls.
- [ ] Implement `GET /api/v1/passes/apple/:userId` endpoint returning binary `.pkpass`.
- [ ] Implement `GET /api/v1/passes/google/:userId` endpoint returning the JWT redirection URL.
- [ ] Integrate Prisma to query user, roles, and company affiliations in `elite_pass_db`.

## Phase 4: Nginx & PM2 Deployment
- [ ] Create `ecosystem.config.js` to manage the process via PM2.
- [ ] Add the service to PM2 and ensure automatic restart on reboot.
- [ ] Update Nginx configuration ([reservas.genial-it.net](file:///etc/nginx/sites-available/reservas.genial-it.net)) to proxy `/billetera-api/` to `http://localhost:3300`.
- [ ] Test network connectivity and endpoints on the new server.

## Phase 5: Client Frontend Integration
- [ ] Design and add standard "Add to Apple Wallet" and "Add to Google Wallet" buttons.
- [ ] Update Client account views ([mi-cuenta](file:///home/soporte/elitepass-reservas/src/app/(external)/mi-cuenta/page.tsx) and [mis-beneficios](file:///home/soporte/elitepass-reservas/src/components/shared/mis-beneficios/mis-beneficios-container.tsx)) to fetch these endpoints.
- [ ] Perform end-to-end checks on iOS and Android devices.

## Phase 6: Alternative Solution (PWA Virtual Card)
- [x] Implement local storage caching (stale-while-revalidate fallback) in [VirtualCredentialDialog](file:///home/soporte/elitepass-reservas/src/components/shared/virtual-credential/virtual-credential-dialog.tsx) to allow loading card details offline.
- [x] Add runtime caching rule for images in [sw.ts](file:///home/soporte/elitepass-reservas/src/app/sw.ts) to cache user avatars/logos locally.
- [x] Add 3D perspective and card flipping utilities to [globals.css](file:///home/soporte/elitepass-reservas/src/app/globals.css).
- [x] Redesign [VirtualCredentialDialog](file:///home/soporte/elitepass-reservas/src/components/shared/virtual-credential/virtual-credential-dialog.tsx) to render a double-sided, interactive 3D Wallet-style flip card.
- [x] Test and ensure download screenshot feature captures the front card correctly.

