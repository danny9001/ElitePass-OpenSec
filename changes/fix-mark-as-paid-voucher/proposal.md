# Proposal: Integrate MarkAsPaidDialog and Fix Payment Voucher Flow

## Business Intent / Bug Report
The user reported that when pre-approving a request and uploading the payment voucher, the voucher does not get saved/uploaded successfully when marking the request as paid.

Our investigation shows that:
1. `MarkAsPaidDialog` is fully implemented in the components codebase (`manager-action-dialog.tsx`) but was never imported or rendered in the main request listing views (`request-list.tsx`).
2. When managers click "Aprobar" on a pre-approved request that hasn't been marked as paid, the approve action in `ApproveDialog` is disabled, and they get stuck without a way to mark it as paid or upload the voucher from the table view.
3. In `request-card.tsx` and dialogs, state initialization of voucher URLs does not react dynamically to changing request references, potentially causing stale voucher values.

Our goal is to:
1. Render and wire `MarkAsPaidDialog` inside `request-list.tsx`.
2. Automatically route managers to the `MarkAsPaidDialog` when they attempt to approve a pre-approved request that is not yet marked as paid.
3. Ensure state variables for voucher URLs update dynamically when dialogs open or request objects change.
