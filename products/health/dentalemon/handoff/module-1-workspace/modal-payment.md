---
screen: Payment
type: modal
route: "N/A (modal within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Dialog
  - DialogHeader
  - Card
  - Badge
  - Alert
  - Select
  - Input
  - Button
  - Separator
  - Switch
wireframe: wireframes/modal-payment.xml
wireframe_pages: [Default, Empty, Fully Paid]
  - "Default"
  - "Empty"
  - "Fully Paid"
---

## Modal: Payment (of Dental Workspace)

**Route:** N/A (modal within `/workspace/:patientId`)
**POV:** A dentist or staff member processing payment for completed treatments at the end of a patient session.

### User Story

> As a dentist, I want to record payments against an auto-generated invoice — supporting multiple payment methods and partial payments — so that billing is seamlessly integrated into the treatment workflow.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Complete | Primary | Closes the invoice and payment modal | `default` |
| Generate Payment Link | Secondary | Creates a shareable link for online payment (Phase 2 placeholder) | `outline` |
| Pay Exact | Ghost | Auto-fills the Amount field with remaining balance | `ghost` |
| Print Receipt | Ghost | Prints receipt | `ghost` |
| Send via Email | Ghost | Emails receipt to patient | `ghost` |
| Send via SMS | Ghost | SMS receipt to patient | `ghost` |
| ✕ | Ghost | Closes modal (invoice remains open if unpaid) | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container | — |
| `DialogHeader` | "Payment" title + ✕ close | — |
| `Card` | Invoice header: Invoice #, Total, Status badge | — |
| `Badge` | Invoice status: "Open" (green outline) / "Paid" (green filled) | `variant='outline'` |
| `Card` | Applied payment entries (method icon + amount + "Confirmed" badge + attribution) | — |
| `Badge` | "Confirmed" badge on each payment | `variant='outline'` |
| `Alert` | Remaining Balance bar (amber when > 0, green when = 0) | `variant='warning'` or `variant='success'` |
| `Select` | Payment method dropdown: Cash, Credit Card, Bank Transfer, Online | — |
| `Input` | Payment amount field | `type='number'`, `placeholder='₱0.00'` |
| `Button` | "Pay Exact" link | `variant='ghost'` |
| `Button` | "Generate Payment Link" | `variant='outline'` |
| `Separator` | Between add payment and receipt sections | — |
| `Button` | Receipt actions: Print Receipt, Send via Email, Send via SMS | `variant='ghost'` |
| `Button` | "Complete" footer CTA (full-width amber) | `variant='default'` |
| `Switch` | PWD discount toggle (inline, below invoice header) | `label='PWD Discount'` |
| `Switch` | Senior Citizen discount toggle (inline, below invoice header) | `label='Senior Citizen Discount'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Method | select | Yes | Must select from: Cash, Credit Card, Bank Transfer, Online | Cash | `Payment.method` |
| Amount | number | Yes | Must be > 0, ≤ remaining balance. Currency format (₱). | Remaining balance | `Payment.amount` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between`. "Payment" title + ✕ close.
2. **Invoice Card** — `mx-6 p-4 border rounded-lg`. Left: "Invoice #INV-YYYY-MMDD". Right: "Total: ₱X,XXX" + Status Badge (Open/Paid).
3. **Applied Payments** — `mx-6 mt-4 space-y-2`. Header: "Applied Payments". Stack of payment cards: method icon + "Cash" / "Credit Card" + amount + "Confirmed" badge. Attribution line: "Received by: [staff name]".
4. **Remaining Balance** — `mx-6 mt-4 px-4 py-2 rounded-lg`. Amber background when balance > 0. Shows: "Remaining Balance" (left) + "₱X,XXX" (right, bold). Green when balance = 0.
5. **Add Payment** — `mx-6 mt-4 flex gap-4`. Method Select (2/3 width) + Amount Input (1/3 width) + "Pay Exact" link.
6. **Generate Payment Link** — `mx-6 mt-4`. Full-width outline button.
7. **Receipt** — `mx-6 mt-4 pt-4 border-t`. "Receipt" label. Three ghost buttons in row: Print Receipt | Send via Email | Send via SMS.
8. **Footer** — `mx-6 mt-6 mb-4`. "Complete" full-width primary button (amber).

### Key Interactions

- **Modal opens** with auto-generated invoice (INV-YYYY-MMDD format). Total = sum of completed treatments (Work Done checked).
- **Add payment** → select method + enter amount (or "Pay Exact" to auto-fill remaining balance). Payment card added to Applied Payments stack. Remaining Balance decrements.
- **Multi-payment support** → multiple payments can be stacked (e.g., ₱5,000 cash + ₱3,500 credit card). Each appears as a separate card.
- **Remaining Balance bar** → amber when > 0, green "All payments received" when = 0.
- **"Generate Payment Link"** → Phase 1 placeholder. Shows toast: "Online payment coming soon." Phase 2: creates shareable GCash/Maya/card link.
- **Staff attribution** → each payment auto-stamps "Received by: [logged-in user name]".
- **"Complete"** → closes invoice and modal. Status changes to "Paid" (if balance = 0) or stays "Open" (if partial). Remaining balance feeds into patient's overall debt (FR5). **Post-Complete workspace state:** modal closes, workspace resumes. If fully paid: "Continue to Payment" footer CTA disappears. Work Done checkboxes remain interactive (dentist may continue adding treatments in the same session). If partially paid: "Continue to Payment" remains visible showing remaining balance.
- **PWD/Senior discount** → if patient is flagged, discount line appears above the invoice total. Auto-calculated per RA 7277/RA 9994. **Inline PWD/Senior toggles** are available directly in the Payment Modal (below the invoice header) so dentists can set the flag without leaving the workspace. Toggling updates the discount calculation immediately. Changes sync to the patient record (Module 2 Patient Profile reflects the updated flag).
- **Receipt actions** → available after any payment is recorded. **Print:** triggers system print dialog; on success: toast "Receipt sent to printer". On failure (no printer): toast "No printer found." **Email:** prompts for email address (pre-filled from patient record if available); on success: toast "Receipt sent to [email]". On failure: toast "Failed to send — check email address." **SMS:** prompts for phone number (pre-filled from patient record); on success: toast "Receipt sent to [number]". On failure: toast "Failed to send — check number." Modal remains open after all receipt actions.
- **Auto-send receipt (FR14.1)** → when Settings > Notifications > Email Receipts toggle is ON, a receipt email is automatically sent to the patient's email address on file after each payment is recorded. Toast: "Receipt auto-sent to [email]." If no email on file: toast "Auto-receipt skipped — no email address on patient profile." The manual receipt buttons (Print/Email/SMS) remain available regardless of the auto-send toggle.
- **Closing without paying** → invoice stays "Open". Patient's debt balance updated.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Invoice card shimmer. Applied Payments empty. |
| **Empty (no payments yet)** | Invoice shows total. No applied payments. Remaining Balance = Total. Add Payment form ready. |
| **Fully paid** | Remaining Balance bar green: "All payments received" + timestamp. "Complete" button enabled. |
| **Error (payment recording fails)** | Toast: "Payment failed to record" + Retry. Modal stays open. |

### CRUD Interactions

#### Adding (payment)
- **Trigger:** Select method + enter amount in Add Payment form
- **Opens:** Inline form (no separate modal)
- **Validation:** Amount > 0, ≤ remaining balance. Method required.
- **On success:** Payment card added to Applied Payments. Remaining Balance decrements. "Confirmed" badge shown.
- **On cancel:** Clear form fields.

### What This Modal Does

- Generates auto-numbered invoices from completed treatments
- Records payments with method, amount, and staff attribution
- Supports multi-payment stacking (cash + card + etc.)
- Tracks remaining balance with visual indicators
- Provides receipt delivery (Print, Email, SMS)
- Applies PWD/Senior discounts automatically
- Placeholder for "Generate Payment Link" (Phase 2)

### What This Modal Does NOT Do

- Does NOT process online payments — Phase 2 (GCash/Maya/card via hosted checkout)
- Does NOT manage patient debt history — that is Module 2: Patient Management (FR5)
- Does NOT generate revenue reports — that is Module 4: Reporting & Analytics
- Does NOT allow editing completed treatments — go back to the breakdown table for that

### Wireframe

See [wireframes/modal-payment.xml](wireframes/modal-payment.xml)
