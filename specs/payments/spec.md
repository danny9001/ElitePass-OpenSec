# Payment Methods Functional Specification

## Scenario: Company Configures Payment Accounts
**Given** an admin user of a company (e.g., JET) is logged into the platform
**When** they navigate to their company settings and the "Payment Accounts" section
**Then** they can add details for multiple accounts (e.g., BNB, BCP, custom QR)
**And** they can mark one of them as their global default account if desired

## Scenario: Event Payment Strategy Selection
**Given** a company has configured an automated payment gateway and/or manual payment accounts
**When** an admin creates or edits an Event
**Then** they can explicitly select the Payment Collection Strategy (Estrategia de Cobro QR) for the event
**And** they can choose either "Automated Payment Gateway" or "Manual QR (Cuentas Bancarias)"
**And** if they choose "Manual QR", they can enable which accounts apply to this specific event and select ONE default account

## Scenario: POS System Default Payment Method
**Given** an event is configured with a payment strategy
**When** a cashier processes a sale in the POS system for this event
**And** the cashier selects a bank/QR payment method
**Then** the POS system automatically pre-selects the event's default strategy (either the Automated Gateway or the specific Manual QR account)

## Scenario: Reservations Default Payment Method
**Given** an event is configured with a default payment strategy (e.g., BNB Gateway or specific Manual Account)
**When** a user makes a reservation for this event
**Then** the reservation checkout screen defaults to showing the configured payment details/QR flow

## Scenario: Platform Default Accounts
**Given** a company has no custom payment accounts configured
**When** creating an event or making a payment
**Then** the system should fallback to a platform-level default account configured globally.
