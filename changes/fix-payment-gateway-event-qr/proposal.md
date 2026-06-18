# Proposal: Payment Gateway Config Fix & Event QR Strategy

## Business Intent
The user reported two issues regarding payment processing:
1. When configuring the automated Payment Gateway for an Empresa, the system throws validation errors (`merchantCode` too small, `apiKey` invalid/missing) if the API key is not provided on subsequent saves (since it's masked).
2. When creating or editing an Event in the reservation system, there is no explicit way to select whether the event will use the Automated Payment Gateway or the Manual QR (Bank Accounts) strategy.

## Goal
Fix the payment gateway configuration update bug so that users can update other configuration parameters without re-entering the API key. In addition, introduce an explicit selection mechanism during Event Creation/Edition to assign the payment collection strategy: either Automated Gateway or Manual QR.
