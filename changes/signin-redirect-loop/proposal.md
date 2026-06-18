# Proposal: Fix Sign-In Redirect Loop for External Users (/mi-cuenta)

## Business Intent / Bug Report
When a user attempts to access their account profile via `https://reservas.genial-it.net/sign-in?redirect=/mi-cuenta`, they are repeatedly redirected back to the login page and are unable to access `/mi-cuenta`.

## Goal
Ensure that customers (external users) logging in through the unified ElitePass Identity service are correctly identified with the `"EXTERNAL"` role in the reservations application so that they can access the `/mi-cuenta` page without redirect loops.
