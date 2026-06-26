# Proposal: QA Connection and Endpoint Verification Script

## Business Intent
ElitePass requires automated verification that the databases for both POS and Reservas are connected, fully functional, capable of creating and querying records, and that the main public health API endpoints are up and returning 200 OK. This ensures system sanity, database integrity, and routing correctness.

## Scope
1. **POS DB Verification**:
   - Establish a connection to the POS database.
   - Fetch the first `Empresa` and `Usuario`.
   - Create a mock product: `Test QA Beer` with category `BEBIDA_ALCOHOLICA` and `precioBase: 15.0` linked to the fetched `Empresa`.
   - Verify that the created product can be queried successfully.
2. **Reservas DB Verification**:
   - Establish a connection to the Reservas database.
   - Fetch the first `Organization` and `User`.
   - Create a mock event: `Test QA Club Night` scheduled for one week from today under the fetched `Organization` (setting required visibility windows).
   - Verify that the created event can be queried successfully.
3. **Public API Endpoints Verification**:
   - Perform HTTP requests to the public health endpoints (e.g. `/api/health` or `/api/v1/health`) of both POS and Reservas servers, ensuring a 200 OK response.
