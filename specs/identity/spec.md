# Functional Specification: User Password and Bulk Actions in Identity Dashboard

## Overview
Administrative users (Admins and Superadmins) in `elitepass-identity` must be able to change user passwords individually and perform bulk actions (bulk password updates and bulk deletions) directly from the dashboard. All actions must enforce role checks, validation rules, safe transaction handling, and proper audit trails.

## Scenarios

### Scenario 1: Admin Changes Password for an Individual User
- **Given** an authenticated user with role `"admin"` or `"superadmin"` is on the user list or user detail page
- **When** they click "Cambiar contraseña" for user `john.doe@example.com`
- **And** input a valid new password (at least 8 characters)
- **Then** the system hashes the password using `bcrypt` (12 rounds)
- **And** updates the credential password for that user in the `Account` table (or creates it if the user was originally registered via another provider but now needs credential access)
- **And** writes an audit log event `"user_password_changed"`
- **And** revalidates the UI paths to display the changes

### Scenario 2: Individual Password Too Short
- **Given** an admin is changing a user's password
- **When** they input a password shorter than 8 characters
- **Then** the system rejects the request with an error message: `"La contraseña debe tener al menos 8 caracteres."`

### Scenario 3: Admin Performs Bulk Password Reset
- **Given** an admin has selected multiple users in the Personas Table
- **When** they click "Cambiar contraseña en bulk"
- **And** input a valid password (at least 8 characters)
- **Then** the system hashes the password using `bcrypt` (12 rounds)
- **And** runs a database transaction updating/creating the credential accounts for all selected user IDs
- **And** logs a single audit event `"bulk_user_password_changed"` containing the count and IDs of affected users

### Scenario 4: Admin Performs Bulk Deletion
- **Given** an admin has selected multiple users in the Personas Table
- **When** they click "Eliminar en bulk" and confirm
- **Then** the system deletes all selected users from the database except for the admin's own user ID (safeguard)
- **And** logs a single audit event `"bulk_user_deleted"` containing the count and IDs of deleted users
