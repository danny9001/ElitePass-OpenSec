# Design: Fix Sidebar Visibility and Dynamic App Icon/Logo in Identity Dashboard

## Technical Decisions

1. **Server-Side Data Propagation**:
   In `src/app/dashboard/layout.tsx`, query the `app` table in the database for the dynamic branding attributes of the `"identity"` app:
   ```typescript
   const identityApp = await db.app.findUnique({
     where: { slug: "identity" },
     select: { nombre: true, iconUrl: true },
   });
   ```
   Pass these values as `appName` and `appIcon` props into `<DashboardShell>`.

2. **Component Interface Extension**:
   - Update `DashboardShellProps` and `SidebarProps` interfaces in `src/components/DashboardShell.tsx` to accept optional `appName` (string) and `appIcon` (string | null) parameters.
   - Fall back to `"ElitePass Identity"` and `null` if they are omitted.

3. **Dynamic Sidebar Header and Mobile Top Bar rendering**:
   Replace the hardcoded `<div className="w-8 h-8 rounded-lg bg-indigo-600 flex items-center justify-center font-bold text-sm shrink-0">EI</div>` template with conditional rendering:
   - If `appIcon` is set, render an `<img src={appIcon} className="w-8 h-8 rounded-lg object-cover shrink-0" />`.
   - Otherwise, fallback to the text initials of the `appName`.

4. **Sticky Sidebar CSS Layout**:
   In `src/components/DashboardShell.tsx`, add `h-screen sticky top-0` classes to the desktop `<aside>` element:
   ```html
   <aside className="hidden lg:flex w-64 h-screen sticky top-0 bg-gray-900 border-r border-gray-800 flex-col shrink-0">
   ```
   This ensures the sidebar spans the viewport height and stays in place while the content area scrolls. The footer containing user profile info and logout remains visible at the bottom of the screen.
