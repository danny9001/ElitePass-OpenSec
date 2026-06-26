# Tasks: Integrate MarkAsPaidDialog and Fix Payment Voucher Flow

- [x] Import `useEffect` and add it to synchronize state in `MarkAsPaidDialog` inside `manager-action-dialog.tsx`.
- [x] Import `MarkAsPaidDialog` and define state `isMarkAsPaidOpen` inside `request-list.tsx`.
- [x] Update `handleApprove` to trigger `MarkAsPaidDialog` when approving an unpaid pre-approved request inside `request-list.tsx`.
- [x] Render `<MarkAsPaidDialog ... />` in `request-list.tsx`.
- [x] Add "Confirmar pago" option to `request-table-view.tsx` dropdown.
- [x] Compile the build using `pnpm build`.
- [x] Reload PM2 cluster `pm2 reload elitepass-reservas`.
- [x] Verify reservation workflow QA verification scripts.
- [x] Bump version in `package.json` to 1.4.94.
- [x] Log results to `MEMORY.md`.
