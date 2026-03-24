---
screen: Revenue Reports
type: screen
route: /reports
module: "Module 4: Reporting & Analytics"
tier: 2
components:
  - DatePicker
  - Button
  - Card
  - Table
  - TableHeader
  - TableRow
  - Separator
  - Badge
wireframe: wireframes/revenue-reports.xml
wireframe_pages: [Default, Empty]
  - Default
  - Empty
---

## Screen: Revenue Reports

**Route:** `/reports`
**POV:** A dentist reviewing practice revenue over a custom date range — tracking collections, treatments performed, and outstanding balances for accounting and tax purposes.

### User Story

> As a dentist, I want to generate date-range revenue reports with treatment breakdowns so that I can track my practice's financial performance and export data for tax filing.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Export CSV | Secondary | Downloads current report as CSV file | `outline` |
| Export PDF | Secondary | Downloads current report as PDF file | `outline` |
| Apply (date range) | Primary | Regenerates report for selected date range | `default` |
| Today | Ghost | Sets date range to today only (daily summary view) | `ghost` |
| This Week | Ghost | Sets date range to current week | `ghost` |
| This Month | Ghost | Sets date range to current month | `ghost` |
| Custom | Ghost | Enables custom start/end date pickers | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `DatePicker` | Start date for custom range | `defaultValue={startOfMonth}` |
| `DatePicker` | End date for custom range | `defaultValue={today}` |
| `Button` | Quick range selectors: Today, This Week, This Month, Custom | `variant='ghost'` |
| `Button` | "Apply" (regenerate report) | `variant='default'` |
| `Button` | "Export CSV" | `variant='outline'` |
| `Button` | "Export PDF" | `variant='outline'` |
| `Card` | Summary metric cards (Total Revenue, Treatments Performed, Patients Seen, Outstanding) | — |
| `Table` | Treatment breakdown table (Treatment Type, Count, Total Revenue, Avg Price) | `aria-label='Treatment Breakdown'` |
| `TableHeader` | Column headers | — |
| `TableRow` | Treatment type rows | `className='h-[52px]'` |
| `Card` | Daily collection summary card (when date range = single day) | — |
| `Separator` | Between summary cards and breakdown table | — |
| `Card` | Daily summary session cards (when Today preset selected): patient name, treatments completed, payment amount, method | — |
| `Badge` | Outstanding balance indicator | `variant='destructive'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Start Date | date | Yes | Must be ≤ End Date | First of current month | — |
| End Date | date | Yes | Must be ≥ Start Date | Today | — |
| Range Preset | button group | No | Today / This Week / This Month / Custom | This Month | — |

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Revenue Reports" heading. Right: "Export CSV" outline + "Export PDF" outline.
2. **Date Range Controls** — `px-6 py-3 flex items-center gap-4`. Quick range buttons (Today | This Week | This Month | Custom) as a button group. When Custom selected: Start Date picker + End Date picker + "Apply" primary button.
3. **Summary Cards** — `px-6 py-4 grid grid-cols-4 gap-4`. Four metric cards:
   - Total Revenue: ₱XX,XXX (sum of all payments in range)
   - Treatments Performed: N (count of treatments marked Done)
   - Patients Seen: N (unique patients with sessions in range)
   - Outstanding: ₱XX,XXX (unpaid balances from invoices in range) with destructive badge if > 0
4. **Treatment Breakdown Table** — `px-6 py-4`. Columns: Treatment Type, Count (how many times performed), Total Revenue (₱), Average Price (₱). Sorted by Total Revenue descending. Footer row: Totals.
5. **Daily Summary** — When date range = single day (Today preset): additional Card below summary showing: time-ordered list of sessions (patient name, treatments, payment amount, payment method). This is the "end-of-day review" view.

### Key Interactions

- **Quick range presets** → Today, This Week, This Month set the date range and immediately regenerate the report. No "Apply" button needed for presets.
- **Custom range** → selecting "Custom" reveals Start/End date pickers. "Apply" button regenerates.
- **Export CSV** → client-side CSV generation. Includes: date range header, summary metrics, treatment breakdown rows. Toast: "Report exported as CSV." File downloads immediately.
- **Export PDF** → triggers browser print-to-PDF dialog with report formatted for A4/Letter. Toast: "Generating PDF..." then auto-download.
- **Daily summary view** → when Today is selected, an additional section appears showing a time-ordered log of the day's sessions (patient name, treatments completed, payment collected). This is the end-of-day review flow from PRD Section 9.4.
- **Outstanding balance link** → tapping the Outstanding summary card navigates to Module 2 Patient List filtered by patients with outstanding balances (if that filter exists) or shows a breakdown of which patients owe what.
- **Data freshness** → report reflects local database state at load time. If a payment is recorded in the workspace (M1) or patient profile (M2) after the report loaded, the user must re-apply/refresh to see updated numbers.
- **Offline** → fully functional offline. All data from local database.

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Outstanding card (tap) | Patient List (filtered by outstanding balance) | `/patients?status=outstanding` | Module 2: Patient Management. Note: M2 Patient List must support `?status=outstanding` as a URL-driven filter that shows only patients with unpaid balances. If this filter is not implemented, the Outstanding card should instead expand an inline breakdown within the Reports screen showing which patients owe what. |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Date range controls render. Summary cards show skeleton numbers. Breakdown table shows shimmer rows. |
| **Empty (no data in range)** | Summary cards show ₱0 / 0 / 0 / ₱0. Breakdown table: "No treatments recorded in this date range." |
| **Empty (new practice, no data ever)** | Same as above + contextual message: "Start seeing patients to generate reports." |
| **Error** | Toast: "Failed to generate report" + Retry. |
| **Staff role** | Export CSV and Export PDF buttons are hidden (not disabled). Staff can view all report data but cannot export. Tooltip on header area: "Export available for Dentist-Owner only." |

### What This Screen Does

- Generates date-range revenue reports (custom range or presets: Today, This Week, This Month)
- Shows summary metrics: total revenue, treatments performed, patients seen, outstanding balances
- Breaks down revenue by treatment type (count, total, average)
- Provides daily collection summary (end-of-day review) when Today is selected
- Exports reports as CSV or PDF for accounting and tax
- Works fully offline from local database

### What This Screen Does NOT Do

- Does NOT show per-patient financial details — that is Module 2 Patient Profile
- Does NOT provide trend visualization or charts — that is the Analytics Dashboard
- Does NOT forecast revenue — Phase 3 (FR23)
- Does NOT show per-dentist attribution — Phase 2 (FR16)
- Does NOT update in real-time — reflects state at load time
