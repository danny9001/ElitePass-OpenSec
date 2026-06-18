# Tasks: Fix Custom App Login Backgrounds and Web Icons

- [x] Update `src/app/globals.css` to exclude `.login-bg-container` from background override rules. <!-- id: 0 -->
- [x] Add `.login-bg-container` class to the outer wrapper `div` in `src/app/login/LoginForm.tsx`. <!-- id: 1 -->
- [x] Add `.login-bg-container` class to the outer wrapper `div` in `src/app/login/mfa/page.tsx`. <!-- id: 2 -->
- [x] Implement `generateMetadata()` in `src/app/layout.tsx` to fetch and apply the custom identity icon as favicon. <!-- id: 3 -->
- [x] Rebuild the project (`pnpm build`) to verify all changes compile cleanly. <!-- id: 4 -->
- [x] Restart `elitepass-identity` on PM2 to apply the updates. <!-- id: 5 -->
