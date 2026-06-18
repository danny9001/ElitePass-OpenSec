# Proposal: Fix Sidebar Visibility and Dynamic App Icon/Logo in Identity Dashboard

## Business Intent
Users navigating the ElitePass Identity dashboard report that the customized application logo/icon is not rendered inside the sidebar and mobile top bars, which instead show a static fallback ("EI"). Additionally, when scrolling through long content lists, the sidebar footer containing the user profile and the "Cerrar sesión" (Logout) button scrolls away and is not always visible on the viewport. This proposal resolves these branding and usability issues.

## Objectives
1. Fetch the custom application name and logo icon URL from the database and feed it to the `DashboardShell` component.
2. Render the custom logo image in place of the static "EI" text block in both the desktop sidebar header and the mobile top bar.
3. Make the desktop sidebar layout sticky (`h-screen sticky top-0`) to keep the navigation and the user profile footer always visible on the viewport regardless of content length.
