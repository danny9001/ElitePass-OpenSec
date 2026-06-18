# Design: Fix Custom App Login Backgrounds and Web Icons

## Root Cause Analysis
1. **Login Background Image**:
   In `src/app/globals.css`, the following rule forces all screens to have the default theme background:
   ```css
   html, body, .min-h-screen, .bg-gray-950, .bg-zinc-950 {
     background-color: var(--background) !important;
     background-image: none !important;
     color: var(--foreground) !important;
   }
   ```
   The wrapper `div` in `LoginForm.tsx` has both `.min-h-screen` and `.bg-gray-950` classes. Since the CSS rule uses `!important`, it completely overrides the custom inline style `backgroundImage` and `backgroundColor` settings generated from the database template.

2. **Browser Icon (Favicon)**:
   `src/app/layout.tsx` defines static layout metadata. It has no mechanism to retrieve the custom logo URL for the Identity app from the database, meaning browsers fall back to the static `src/app/favicon.ico`.

## Technical Decisions
1. **CSS Selector Exemption**:
   Introduce a new class `.login-bg-container` and update `globals.css` to exclude it:
   ```css
   html, body, .min-h-screen:not(.login-bg-container), .bg-gray-950:not(.login-bg-container), .bg-zinc-950:not(.login-bg-container) {
     background-color: var(--background) !important;
     background-image: none !important;
     color: var(--foreground) !important;
   }
   ```

2. **Wrapper Class Addition**:
   Add `.login-bg-container` to the top-level outer divs in:
   - `src/app/login/LoginForm.tsx`
   - `src/app/login/mfa/page.tsx`

3. **Dynamic Favicon Metadata**:
   Update `src/app/layout.tsx` to query the `app` table where `slug = "identity"` to fetch the configured `iconUrl` asynchronously inside the `generateMetadata()` function:
   ```typescript
   export async function generateMetadata(): Promise<Metadata> {
     const identityApp = await db.app.findUnique({
       where: { slug: "identity" },
       select: { iconUrl: true },
     });
     return {
       title: "ElitePass Identity",
       description: "SSO y directorio centralizado del ecosistema ElitePass",
       icons: {
         icon: identityApp?.iconUrl || "/favicon.ico",
       },
     };
   }
   ```
