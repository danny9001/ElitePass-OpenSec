# Design: Integrate MarkAsPaidDialog and Fix Payment Voucher Flow

## Technical Decisions
1. **Dynamic Voucher State Synchronization**:
   - Update `MarkAsPaidDialog` in `manager-action-dialog.tsx` to include `useEffect` targeting `open` and `request` props. When the dialog opens or the active request changes, synchronize the `voucherUrl` state with the request's database `paymentVoucherUrl`.
2. **List-View Integration of MarkAsPaidDialog**:
   - Import `MarkAsPaidDialog` in `request-list.tsx`.
   - Add state variables (`isMarkAsPaidOpen`, `setIsMarkAsPaidOpen`) to manage its visibility.
   - Render `<MarkAsPaidDialog ... />` alongside other manager action dialogs in `request-list.tsx`.
3. **Table View Dropdown Options**:
   - Add a "Confirmar pago" menu option to the actions dropdown in `request-table-view.tsx` for managers when a request's status is `PRE_APPROVED` and it is not yet paid.
4. **Approval Flow Fallback**:
   - Update `handleApprove` in `request-list.tsx`. If a request is `PRE_APPROVED` but `isPaid` is false, open `MarkAsPaidDialog` instead of `ApproveDialog`. Once marked as paid, subsequent approval actions will open `ApproveDialog` with the Approve button enabled.
5. **Compilation & QA Verification**:
   - Run compilation checks (`pnpm build`).
   - Reload PM2 (`pm2 reload elitepass-reservas`).
   - Run `node scripts/qa-verify.mjs` to ensure the platform is healthy.
