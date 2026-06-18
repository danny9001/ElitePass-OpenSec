# Business Proposal: Google Account MFA Setup Re-authentication and Passkey Direct Entry

## Overview & Context
Currently, when a user configures or disables Multi-Factor Authentication (MFA/2FA) in the **ElitePass Identity** dashboard, the application asks them to confirm their password. However, users who sign in via Google OAuth (a social login provider) do not have a password set in the local database. Asking them for a password blocks them from configuring or disabling MFA.

To address this, we need to bypass the password requirement for Google-linked accounts. Instead of asking for a password, we must ask the user to re-authenticate with Google. Once they successfully re-authenticate (which includes validating Google credentials and prompting for MFA if already enabled), they should be allowed to enable or disable MFA directly.

Furthermore, we must ensure that logins completed via **Passkey/FIDO2** provide "direct entry" (ingreso directo) without prompting for MFA, since passkeys are already inherently multi-factor.

## Objectives
1. **Remove Password Constraint for Google Accounts**: Do not ask for password when a Google account user attempts to enable or disable MFA.
2. **Re-authentication Flow**: Require Google users to re-authenticate by logging in with Google again.
3. **Handle Multi-Step Authentication**: In this re-authentication process, ensure that if MFA is already active (i.e. when disabling MFA), the user is prompted for Google, then their MFA code.
4. **Automatic Action Execution**: On return from successful re-authentication, automatically execute the MFA action (enable/generate TOTP QR code, or disable MFA).
5. **Ensure Passkey Direct Entry**: Guarantee that passkey-based logins continue to serve as a direct entry, bypassing any subsequent MFA prompt since WebAuthn acts as a strong multi-factor method by itself.

## Stakeholders
- **UX/UI Developer**: Responsible for updated UI buttons, dialogs, and flow transitions in the profile page.
- **Backend & Security Specialist**: Responsible for configuring Better-Auth to allow passwordless 2FA management and validating session states.
