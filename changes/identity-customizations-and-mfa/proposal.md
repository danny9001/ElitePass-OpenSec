# Business Proposal: Identity UI/UX Customization, Subscriptions, and Security Hardening

## Overview & Context
The **ElitePass Identity** application controls all user access, single sign-on (SSO), and licensing across the Genial-it ecosystem. To prepare this platform for enterprise multi-tenancy and elevate its look and feel, we are introducing a series of branding, user/company logistics, subscription history, and session security improvements.

## Objectives
1. **Branding & Form Customization**: Establish a premium, minimal, and fully customizable login experience per application, including custom backgrounds, logos, form ordering, custom footers, and light/dark theme support.
2. **Data & Logistics Refinement**:
   - Refactor user ("Persona") forms: hide UPN, set Bolivia ("BO") as the default location, allow photo upload, and enable company membership selector.
   - Refactor company ("Empresa") profiles: capture NIT, country, city, address, phone, logo, color palette, and manage memberships/licenses.
   - Refactor licenses: show status (active/expired) and allow editing their validity date.
3. **SKU & Subscription Lifecycle**:
   - Implement structured SKUs with product name, app association, validity period, and status.
   - Support contract subscriptions (Company + SKU, immutable once created, keeping history on deletion).
4. **Super Admin Management**:
   - Create a dedicated super-administrator management view to edit other admin/super-admin profiles, change their passwords, and upload photos.
5. **Security Hardening**:
   - Propose and implement a strict **5-minute inactivity timeout** (dual-layer: client-side event trackers + server-side validation cookie).
   - Propose and design MFA (Multi-Factor Authentication) integration when only email and password are used.

## Stakeholders
- **UX/UI Brand Specialist**: Responsible for the visual system, premium feel, and preview panel.
- **Backend & Connections Specialist**: Responsible for SSO tokens, Azure storage image uploads, and WebP compression.
- **Database Architect**: Responsible for Prisma schema updates, soft-deletes, and migrations.
- **Security Auditor**: Responsible for MFA flow, session timeouts, and audit logging.
- **Business Logistics Specialist**: Responsible for SKU validity, subscriptions, and memberships.

## Telegram Security Alerts & Styled "Sin Acceso" Page
1. **Telegram Notifications**: Inform the administrator via the local Telegram service (`http://localhost:3200`) of any authentication errors, failed login attempts (on `/api/auth/sign-in/*`), or unauthorized attempts to access the administration panel.
2. **Styled Unauthorized Access View**: Replace the generic 403 page on `/sin-acceso` with a zinc-themed, minimal page that displays only the message: *"Sin acceso: No tienes permisos para acceder al panel de administración."*
3. **Recipient Resolution**: Retrieve the administrator's phone number from the database dynamically, falling back to `+59175213449`.

