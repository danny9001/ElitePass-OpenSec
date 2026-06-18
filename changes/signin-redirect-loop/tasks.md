# Tasks: Sign-In Redirect Loop Fix

- [x] Create functional specification `openspec/specs/auth/spec.md` <!-- id: 0 -->
- [x] Modify `src/app/api/auth/identity-callback/route.ts` in `elitepass-reservas` to parse `accountType` and prevent role downgrade. <!-- id: 1 -->
- [x] Compile and verify the project builds successfully. <!-- id: 2 -->
- [x] Increment the package version of `elitepass-reservas` in `package.json`. <!-- id: 3 -->
- [x] Restart `elitepass-reservas` using PM2. <!-- id: 4 -->
- [x] Commit and push changes. <!-- id: 5 -->
- [x] Update functional specification `openspec/specs/auth/spec.md` with signed cookies scenario. <!-- id: 6 -->
- [x] Modify `src/app/api/auth/identity-callback/route.ts` to sign the session token cookie using `makeSignature`. <!-- id: 7 -->
- [x] Rebuild and compile `elitepass-reservas`. <!-- id: 8 -->
- [x] Increment the package version to `1.4.83`. <!-- id: 9 -->
- [x] Restart the application using PM2. <!-- id: 10 -->
- [x] Commit and push new changes. <!-- id: 11 -->
