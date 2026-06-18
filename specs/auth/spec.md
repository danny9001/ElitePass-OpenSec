# Functional Specification: Ecosistema SSO & Roles Mapping

## Overview
Authentication in the ElitePass ecosystem is standardized via `elitepass-identity` SSO. When logging into an application like `elitepass-reservas`, the authentication callback must map the user attributes from the Identity JWT to application-specific roles correctly and issue signed session cookies that comply with Better Auth security standards.

Additionally, this specification covers two-factor authentication (MFA) configuration behaviors for Google OAuth users and MFA bypasses for Passkey authentication.

## Scenarios

### Scenario 1: External Customer Login
- **Given** a user is classified as `EXTERNAL` or `CLIENT` in `elitepass-identity`
- **When** they login via the SSO callback in `elitepass-reservas`
- **Then** their role in the reservations database is set to `"EXTERNAL"`
- **And** they are allowed access to `/mi-cuenta`

### Scenario 2: Role Preservation
- **Given** a user already has the role `"EXTERNAL"` in the reservations database
- **When** they log in via the SSO callback
- **Then** their role is preserved and not downgraded to `"USER"` (even if they have no explicit reservations license in identity)

### Scenario 3: Admin / Staff Override
- **Given** a user is logging in via SSO
- **When** the identity token explicitly defines a staff role (e.g. `"ADMIN"`, `"SUPER_ADMIN"`, `"STAFF"`) for the `"reservas"` application
- **Then** their role is updated to match the administrative role

### Scenario 4: Secure Cookie Signing
- **Given** the reservations application requires signed session cookies in production
- **When** a session is created during the SSO callback
- **Then** the cookie value must be signed using the application's `BETTER_AUTH_SECRET`
- **And** formatted as `<session_token>.<signature>` to prevent validation failures

### Scenario 5: MFA Configuration for Google Accounts (Re-authentication)
- **Given** a user is logged in via their Google account and has no password
- **When** they attempt to enable or disable Multi-Factor Authentication (MFA) in their profile
- **Then** the system must not request a password confirmation
- **And** it must require the user to re-authenticate with Google
- **And** upon return from a successful Google re-authentication, the MFA setup or disabling must execute automatically

### Scenario 6: MFA Bypass for Passkey Authentication (Direct Entry)
- **Given** a user has Multi-Factor Authentication (MFA) enabled on their account
- **When** they log in using a Passkey (FIDO2)
- **Then** the system must bypass the MFA verification prompt and issue a session immediately
- **And** grant direct entry to the dashboard
