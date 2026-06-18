# Proposal: Align Identity Design and Colors with Reservas

## Business Intent
It was decided that all applications in the ElitePass ecosystem must share the same design language, layout style, buttons, and color palette as defined by `elitepass-reservas`. 
The `elitepass-identity` application (SSO/IAM) currently uses a separate, forced light-theme style override that does not cleanly integrate the oklch-based color variables, system fonts, and 0.625rem (10px) border radii of the Reservas application. This task aims to synchronize the theme and style of the Identity application to match the Reservas style exactly.

## Scope of Changes
- Define the same CSS variables for `:root` and `.dark` in `elitepass-identity/src/app/globals.css` as in `elitepass-reservas`.
- Configure the theme mappings in `@theme inline` inside `globals.css` to align with the Reservas configuration.
- Implement robust CSS overrides in `globals.css` to translate the codebase's hardcoded utility colors (like `bg-gray-950`, `bg-gray-900`, `bg-gray-800`, `bg-indigo-600`, etc.) to Reservas' semantic CSS variables (`var(--background)`, `var(--card)`, `var(--secondary)`, `var(--primary)`, etc.).
- Update layout elements and ensure the system fonts are used (`"Segoe UI", "Helvetica Neue", Helvetica, Arial, system-ui, sans-serif`).
