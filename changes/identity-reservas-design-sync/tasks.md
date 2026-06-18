# Tasks: Identity Design and Colors Sync with Reservas

- [x] Define oklch CSS variables in `elitepass-identity/src/app/globals.css` matching `elitepass-reservas`.
- [x] Configure `@theme inline` in `globals.css` with the Reservas theme attributes.
- [x] Implement `@layer base` style resets for HTML, body, and font.
- [x] Implement overrides in `globals.css` matching hardcoded classes in TSX components to reservoirs theme variables:
  - [x] Page and app backgrounds (`bg-gray-950`, `bg-zinc-950`) map to `var(--background)`.
  - [x] Sidebars and card backgrounds (`bg-gray-900`, `bg-zinc-900`) map to `var(--card)`.
  - [x] Secondary items, inputs and inner elements (`bg-gray-800`, `bg-zinc-800`) map to `var(--secondary)`.
  - [x] Borders (`border-gray-800`, `border-gray-700`, `border-zinc-800`, `border-zinc-700`, etc.) map to `var(--border)`.
  - [x] Texts (`text-white`) map to `var(--foreground)` (with button/badge exceptions).
  - [x] Descriptions/muted text (`text-gray-400`, `text-zinc-400`, `text-gray-500`, `text-zinc-500`) map to `var(--muted-foreground)`.
  - [x] Primary buttons (`bg-indigo-600`, `bg-indigo-700`, button.bg-indigo-600) map to `var(--primary)` and hover states to darker shade.
  - [x] Active links and highlights (`bg-indigo-600/20`, `text-indigo-300`) map to `var(--sidebar-accent)` and `var(--sidebar-accent-foreground)`.
  - [x] Layout headers map to `var(--sidebar)`.
  - [x] Border radius override (`rounded-lg`, `rounded-xl`, `rounded-2xl`) map to `var(--radius)` (10px).
- [x] Build and test the Identity application using `pnpm build` to verify there are no compilation/bundling errors.
- [x] Restart `elitepass-identity` on PM2 and confirm it runs properly.
