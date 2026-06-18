# Proposal: Add Sorting and Filtering Modal to Identity Personas Page

## Business Intent
Administrators of the ElitePass Identity SSO system need to efficiently manage large sets of users. The current interface only allows basic searching by name/email/CI and a simple company filter. This proposal adds a comprehensive sorting and filtering modal to filter users by application, email, company, registration date, and user type, as well as sort them to quickly locate specific demographics.

## Objectives
1. Implement a client-side sorting and filtering modal component.
2. Allow filtering by:
   - Application (App licenses associated with the user's company).
   - User Type (`accountType`: `USER`, `CLIENT`, `STAFF`, `EXTERNAL`).
   - Registration Date range (Start and End dates).
   - Company (integrating/replacing the current simple dropdown).
3. Allow sorting (ascending/descending) by:
   - Registration Date (`createdAt`).
   - Name (`name`).
   - Email (`email`).
4. Update the server-side queries in Next.js `PersonasPage` to process these filters.
5. Provide a clear indicator of how many active filters are applied, and a quick "Reset" action.
