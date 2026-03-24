---
screen: Payment History
type: tab
route: null
module: "Module 2: Patient Management"
tier: 1
components:
  - DatePicker
  - Select
  - Table
  - TableHeader
  - TableRow
  - Badge
  - Button
wireframe: inline-ascii
---

## Tab: Payment History (of Patient Profile)

**Route:** N/A (tab within `/patients/:id`)
**POV:** A dentist reviewing a patient's complete payment history to track debts, verify past payments, or prepare for a billing conversation.

### User Story

> As a dentist, I want to filter and review a patient's payment history by date range and method so that I can track outstanding debts and verify past transactions.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Filter (date range) | Secondary | Applies date range filter to payment list | `outline` |
| Clear Filters | Ghost | Removes all filters, shows full history | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `DatePicker` | Start date filter | `defaultValue={thirtyDaysAgo}` |
| `DatePicker` | End date filter | `defaultValue={today}` |
| `Select` | Payment method filter (All / Cash / Credit Card / Bank Transfer / GCash / Maya / Manual) | `defaultValue='All'` |
| `Table` | Payment history table | `aria-label='Payment History'` |
| `TableHeader` | Column headers: Date, Invoice #, Amount, Method, Treatments, Staff, Status | — |
| `TableRow` | Individual payment rows | `className='h-[52px]'` (touch target) |
| `Badge` | Payment status: "Confirmed" (green) / "Manual" (amber) | `variant='outline'` |
| `Badge` | Method badge per row | `variant='outline'` |
| `Button` | "Clear Filters" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Start Date | date | No | Must be ≤ End Date | 30 days ago | — |
| End Date | date | No | Must be ≥ Start Date | Today | — |
| Method Filter | select | No | — | All | — |

### Layout

1. **Filter Bar** — `flex gap-4 items-end py-4`. Start Date picker + End Date picker + Method Select + "Clear Filters" ghost button.
2. **Payment Table** — `mt-4`. Columns: Date (formatted), Invoice # (linked to invoice if available), Amount (₱ formatted), Method (badge), Treatments (brief list of associated treatments), Staff ("Received by: [name]"), Status (Confirmed/Manual badge).
3. **Summary Row** — Below table: "Total: ₱X,XXX" for the filtered period. "Outstanding: ₱X,XXX" if unpaid invoices exist.
4. **Empty State** — "No payments recorded" or "No payments match your filters" + "Clear Filters" link.

### Key Interactions

- **Date range filter** → filters payment table to show only payments within the selected date range. Applied on change (no separate "Apply" button needed).
- **Method filter** → further narrows by payment method. Combinable with date range.
- **"Clear Filters"** → resets to defaults (last 30 days, all methods).
- **Invoice # link** → if the invoice still exists, tapping it shows the invoice details (future enhancement — Phase 1 just displays the number).
- **Treatments column** → brief list of treatment names associated with each payment. Truncated with "..." if more than 3.
- **Manual payments** → flagged with "Manual" amber badge (recorded outside the workspace Payment Modal, via the Patient Profile "Record Payment" action).

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Shimmer rows in table. Filter bar renders immediately. |
| **Empty (no payments ever)** | "No payments recorded for this patient." No filter bar shown. |
| **Empty (no filter matches)** | "No payments match your filters." + "Clear Filters" link. |
| **Error** | Toast: "Failed to load payment history" + Retry. |

### What This Tab Does

- Shows a filterable table of all payments for this patient across all encounters
- Supports date range and method filtering
- Distinguishes between confirmed (workspace) and manual (profile) payments
- Shows associated treatments per payment
- Displays summary totals for the filtered period

### What This Tab Does NOT Do

- Does NOT process new payments — use "Record Payment" on the Overview tab or Payment Modal in the workspace
- Does NOT generate reports or exports — that is Module 4: Reporting & Analytics
- Does NOT show payments for other patients — scoped to the current patient only

### Wireframe

```
┌──────────────────────────────────────────────────┐
│ From [03/01/26] To [03/24/26]  Method [All ▾]   │
│                                    Clear Filters  │
├──────────────────────────────────────────────────┤
│ Date       Invoice     Amount  Method  Staff  St │
│──────────────────────────────────────────────────│
│ 03/24/26  INV-0312    ₱5,000  Cash   FDesk  ✓  │
│ 03/24/26  INV-0312    ₱3,500  Card   FDesk  ✓  │
│ 03/12/26  INV-0228      ₱800  Cash   FDesk  ✓  │
│ 03/01/26  (manual)    ₱2,000  GCash  Owner  M  │
│──────────────────────────────────────────────────│
│                  Total: ₱11,300                   │
│                  Outstanding: ₱1,100              │
└──────────────────────────────────────────────────┘
```
