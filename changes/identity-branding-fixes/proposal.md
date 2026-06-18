# Proposal: Fix Custom App Login Backgrounds and Web Icons

## Business Intent
Organizations customizing their SSO application templates via the ElitePass Identity dashboard expect their branding settings—specifically the login background image/color and application logo/favicon—to render correctly. Currently, global CSS overrides prevent custom login backgrounds from showing, and the browser favicon is static. This proposal fixes the CSS selector priorities to allow custom backgrounds on login views and dynamically applies the Identity application's custom icon as the favicon.

## Objectives
1. Fix the CSS background overrides in `globals.css` to exempt the login container class.
2. Apply the `.login-bg-container` class to the main login form and double factor authentication wrapper divs.
3. Dynamically set the web browser favicon in Next.js `RootLayout` using `generateMetadata` to fetch the Identity app's database-configured icon.
4. Verify branding elements render as expected.
