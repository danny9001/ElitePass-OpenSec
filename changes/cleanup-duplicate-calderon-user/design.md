# Design: Clean Up Duplicate Alejandra Calderon User Account

## Root Cause Analysis
During registration, the user initially created an account under the email `acladeron@dima.com.bo` (Allejandra Calderon, id: `a8ls3fioda9uhdypzhhr6clt`) on `2026-06-11 16:20:04.906`. Four minutes later, on `2026-06-11 16:24:42.079`, the user created another account under the correct email `acalderon@dima.com.bo` (Alejandra Calderon, id: `p6ej1tst84etx22g1bz3b4sa`).
The second account (`acalderon@dima.com.bo`) is the one with 42 active/expired sessions and recent updates, while the first account (`acladeron@dima.com.bo`) has no session history and remains unused.

## Technical Decisions
1. **Target Account for Deletion**:
   We will delete `acladeron@dima.com.bo` (id: `a8ls3fioda9uhdypzhhr6clt`) from the `user` table in the `elitepass_identity` database.

2. **Cascading Deletion**:
   Since the `account` table in `elitepass_identity` has a foreign key constraint `account_userId_fkey` with `ON DELETE CASCADE` referencing the `user` table, deleting the user will automatically clean up the associated record in `account`.

3. **Verification**:
   - Query the `user` and `account` tables before deletion to confirm the target data.
   - Run the delete SQL statement.
   - Query the `user` and `account` tables after deletion to confirm the duplicate is gone and the active user (`acalderon@dima.com.bo`) remains unaffected.
