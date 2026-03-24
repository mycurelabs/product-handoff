---
screen: Treatment History
type: modal
route: "N/A (modal within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Dialog
  - DialogHeader
  - Table
  - Badge
  - Separator
  - ScrollArea
  - Select
  - Button
wireframe: inline-ascii
---

## Modal: Treatment History (of Dental Workspace)

**Route:** N/A (modal within `/workspace/:patientId`)
**POV:** A dentist reviewing the patient's complete treatment history across all encounters — a comprehensive record view accessible from any baseline.

### User Story

> As a dentist, I want to view all treatments, conditions, and plans from every encounter in one place so that I have a complete clinical picture of the patient's dental history.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| ✕ | Ghost | Closes modal, returns to workspace | `ghost` |
| Clear Filters | Ghost | Resets all filters to default (All/All Time/All Teeth) | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container (centered, large) | `className='max-w-[800px]'` |
| `DialogHeader` | "Treatment History" title + ✕ close | — |
| `Table` | Treatment rows grouped by encounter date | — |
| `Badge` | Status badge per row: "Done" (green), "Proposed" (amber), "Paid" (green filled) | `variant='outline'` |
| `Separator` | Between encounter date groups | — |
| `ScrollArea` | Scrollable content area for large histories | — |
| `Select` | Status filter: All / Proposed / Done / Paid | `defaultValue='All'` |
| `Select` | Date range filter: All Time / Today / This Week / This Month / Custom | `defaultValue='All Time'` |
| `Select` | Tooth filter: All Teeth / specific tooth number | `defaultValue='All Teeth'` |
| `Button` | Clear Filters | `variant='ghost'` |

### Layout

1. **Header** — "Treatment History" + ✕ close button.
2. **Filter Bar** — `flex gap-3 py-2`. Status filter (All/Proposed/Done/Paid) + Date range filter (All Time/Today/This Week/This Month/Custom) + Tooth filter (All Teeth / tooth number — pre-selected when opened from Slideout Overview "View All"). "Clear Filters" ghost button.
3. **Content** — Scrollable list of treatments grouped by encounter date (most recent first), filtered by active filters. Each group has a date header (e.g., "March 24, 2026"). Under each date: treatment rows — Tooth · Surface, Condition → Treatment, Status badge (Done/Proposed/Paid), Price. Rows are read-only. Muted text for past encounters.
4. **Summary Footer** — Separator + totals reflect filtered results: "Total Paid: ₱X,XXX" | "Outstanding: ₱X,XXX".

### Key Interactions

- **Read-only** — no actions on rows. This is a review/reference view.
- **Grouped by encounter date** — most recent encounter at top. Each group shows the encounter's treatments.
- **Same modal regardless of carousel position** — whether viewing today's baseline or a past one, "View All" always shows the complete history.
- **Status badges** reflect current status: "Proposed" (planned, not done), "Done" (completed, not yet paid), "Paid" (completed and paid).
- **Outstanding balance** — if any treatments across encounters remain unpaid, the footer shows the aggregate outstanding amount.
- **Status filter** — filters treatment rows by status. "All" shows everything. "Proposed" shows only planned treatments. "Done" shows completed-unpaid. "Paid" shows paid treatments. Footer totals recalculate to reflect filtered results.
- **Date range filter** — filters by encounter date. "All Time" shows full history. Presets: Today, This Week, This Month. "Custom" reveals a date range picker.
- **Tooth filter** — filters by tooth number. "All Teeth" shows everything. Selecting a specific tooth shows only treatments for that tooth. **Pre-selected when opened from Slideout Overview "View All"** — if the dentist opened Treatment History from Tooth #16's Overview step, the Tooth filter defaults to "16" instead of "All Teeth."
- **Clear Filters** — resets all 3 filters to defaults (All / All Time / All Teeth). Hidden when all filters are already at default.
- **Empty filter results** — when active filters produce 0 results: "No treatments match the current filters." + "Clear Filters" link. Footer totals show ₱0 / ₱0.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Shimmer rows grouped by date. |
| **Empty (no history)** | "No treatment history for this patient." (rare — only for brand-new patients with no wizard completions). |
| **Empty (filter, no results)** | "No treatments match the current filters." + "Clear Filters" link. |
| **Error** | N/A — reads from local state. |

### What This Modal Does

- Shows ALL treatments from ALL encounters for the patient in one scrollable view
- Groups treatments by encounter date (most recent first)
- Displays status per treatment (Proposed / Done / Paid)
- Filters by status, date range, and tooth number
- Shows aggregate totals (paid + outstanding) that update with active filters
- Pre-selects tooth filter when opened from a specific tooth's Overview

### What This Modal Does NOT Do

- Does NOT allow editing or removing treatments — go back to the breakdown table for the relevant baseline
- Does NOT process payments — that is the Payment Modal
- Does NOT show treatments from other patients — scoped to the current patient only

### Wireframe

```
┌──────────────────────────────────────────────────────┐
│ Treatment History                                 ✕  │
├──────────────────────────────────────────────────────┤
│ [All ▾]  [All Time ▾]  [All Teeth ▾]  Clear Filters │
├──────────────────────────────────────────────────────┤
│                                                      │
│ March 24, 2026                                       │
│ ──────────────────────────────────────────────────── │
│ #9 · M,D   Caries → Composite Fill.    Done    ₱800 │
│ #9 · M,D   Fractured → Cleaning        Done    ₱800 │
│ #1 · M     Fractured → Cleaning     Proposed   ₱300 │
│                                                      │
│ March 12, 2026                                       │
│ ──────────────────────────────────────────────────── │
│ #14 · B    Caries → Filling             Paid    ₱500 │
│ #22 · O    Caries → Sealant             Paid    ₱300 │
│                                                      │
│ February 28, 2026                                    │
│ ──────────────────────────────────────────────────── │
│ #3 · All   Impacted → Extraction        Paid  ₱2,500│
│                                                      │
│──────────────────────────────────────────────────────│
│ Total Paid: ₱4,100          Outstanding: ₱1,100     │
└──────────────────────────────────────────────────────┘
```
