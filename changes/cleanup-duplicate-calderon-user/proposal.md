# Proposal: Clean Up Duplicate Alejandra Calderon User Account

## Business Intent
A user registered twice under slightly different email spellings: `acladeron@dima.com.bo` and `acalderon@dima.com.bo`. To maintain data integrity, prevent authentication confusion, and ensure the correct user profile is used, we need to remove the incorrect/older user account and keep only the latest active user account.

## Objectives
1. Identify and verify both duplicate user accounts in the SSO database (`elitepass_identity`).
2. Determine which user account is the active one with recent login history.
3. Securely delete the duplicate/incorrect user account (`acladeron@dima.com.bo`) and its cascading relations.
4. Verify that the correct user account (`acalderon@dima.com.bo`) remains intact and can access the application without issues.
