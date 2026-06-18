# Design: Identity Design and Colors Sync with Reservas

## Analysis & Current State
- `elitepass-identity` uses Tailwind CSS v4.
- In `src/app/globals.css`, there are overrides under the comment `FORCE LIGHT THEME OVERRIDES FOR DARK UTILITIES` which override classes like `bg-gray-950`, `bg-gray-900`, `bg-gray-800`, `border-gray-800`, etc. with hardcoded hex colors (`#f9fafb`, `#ffffff`, `#e5e7eb`, etc.).
- `elitepass-reservas` has a cohesive oklch-based color palette mapped in `globals.css` with `:root` and `.dark` variables, and custom theme inline definitions.
- The standard ecosystem style requires:
  - System font family: `"Segoe UI", "Helvetica Neue", Helvetica, Arial, system-ui, sans-serif`
  - Border radius: `0.625rem` (10px)
  - Unified oklch variables for card, background, borders, primary, secondary, accent, etc.

## Technical Decisions
1. **Copy Theme & Variables:** Define the exact color variables and `@theme inline` from `elitepass-reservas` inside `elitepass-identity/src/app/globals.css`.
2. **System Fonts:** Set the standard system font stack.
3. **Map Overrides to Variables:** Instead of hardcoded colors in CSS overrides, map the classes in the overrides to the semantic CSS variables (`var(--background)`, `var(--card)`, `var(--secondary)`, `var(--border)`, `var(--primary)`, `var(--radius)`, etc.).
4. **Sidebar & Layout Harmonization:** Map the sidebar active state backgrounds and texts, as well as the main header, to the ecosystem `var(--sidebar)` and `var(--sidebar-accent)` variables.
5. **Button Radius Harmonization:** Map all `.rounded-lg`, `.rounded-xl`, and `.rounded-2xl` classes to use `var(--radius)` so that all buttons, form inputs, and cards feature a consistent border radius of 10px.
