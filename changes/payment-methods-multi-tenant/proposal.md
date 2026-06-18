# Proposal: Multi-Tenant Payment Methods (Reservations & POS)

## Business Intent
The current system does not allow tenants (companies) to configure their own custom payment methods/accounts (e.g., BCP, BNB, custom QR codes).
When an Event is created or edited, or when a POS sale is processed, the platform must allow specifying which payment account is used by default.

### Key Requirements
1. **Per-Company Configuration**: A microservice/backend endpoint must allow each company to register multiple payment accounts (BNB, BCP, manual QR).
2. **Platform Default**: The platform can provide default payment accounts for companies that don't have their own.
3. **Event Association**: When creating or editing an Event, the user must be able to select the payment methods enabled for that event, and choose a default one.
4. **Reservations & POS**: Both the reservations module and the POS system must use the payment methods configured for the event. The default payment method chosen during event creation will be used as the default in reservations and POS.
