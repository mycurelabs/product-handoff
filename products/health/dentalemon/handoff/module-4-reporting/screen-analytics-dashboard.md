---
screen: Analytics Dashboard
type: screen
route: /dashboard
module: "Module 4: Reporting & Analytics"
tier: 2
components:
  - Card
  - Tabs
  - TabsTrigger
  - Badge
  - Button
  - Alert
wireframe: wireframes/analytics-dashboard.xml
wireframe_pages: [Default, Empty, Solo Tier]
  - Default
  - Empty
  - Solo Tier
---

## Screen: Analytics Dashboard

**Route:** `/dashboard`
**POV:** A dentist reviewing practice performance trends — weekly, monthly, and quarterly patterns in revenue, patient volume, and treatment frequency.

### User Story

> As a dentist, I want to visualize my practice's trends over time so that I can identify my most profitable treatments and track growth.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Weekly / Monthly / Quarterly | — | Time period toggle (segmented control) | — |
| View Detailed Report | Ghost | Navigates to Revenue Reports screen | `ghost` |

### ShadCN Block

**Base block:** `dashboard-01` (Dashboard)
**Install:** `npx shadcn@latest add dashboard-01`
**Coverage:** `customized`
**Stock components included:** Card, Chart (Bar), Chart (Line)

### ShadCN Components

| Δ | Component | Usage | Props/Variant |
|---|-----------|-------|---------------|
| `~` | `Card` | Metric summary cards (Revenue, Patients, Treatments, Outstanding) — same metrics as Reports but with trend indicator (↑/↓ %) | Modified: added trend badge |
| `+` | `Tabs` | Time period toggle: Weekly / Monthly / Quarterly | `defaultValue='monthly'` |
| `+` | `TabsTrigger` | Weekly | `value='weekly'` |
| `+` | `TabsTrigger` | Monthly | `value='monthly'` |
| `+` | `TabsTrigger` | Quarterly | `value='quarterly'` |
| `+` | `Badge` | Trend indicator on metric cards: ↑12% (green) or ↓5% (red) vs previous period | `variant='outline'` |
| `+` | `Button` | "View Detailed Report" link to Revenue Reports | `variant='ghost'` |
| `+` | `Alert` | Practice tier upgrade prompt (Solo tier users) | `variant='warning'` |

### ShadCN Charts

| Chart Type | Data Source | Purpose | Config Notes |
|------------|------------|---------|-------------|
| Line | `Payment` records aggregated by period | Revenue trend over time | Multi-series: Collections (primary amber) + Outstanding (destructive red). X-axis: time periods. Y-axis: ₱ amount. Tooltip: period label + exact ₱ value. |
| Bar | `TreatmentRecord` aggregated by treatment type | Top treatments by frequency and revenue | Stacked or grouped: frequency (count) + revenue (₱). Sorted by revenue descending. Top 10 treatments. Horizontal orientation for readability on tablet. |
| Pie | `Payment` records grouped by method | Revenue breakdown by payment method | Segments: Cash, Credit Card, Bank Transfer, GCash/Maya (placeholder), Manual. Labels: method + percentage + ₱ amount. |

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Dashboard" heading. Right: "View Detailed Report" ghost link → `/reports`.
2. **Time Period Toggle** — `px-6 py-2`. Weekly / Monthly / Quarterly segmented control (Tabs).
3. **Summary Cards** — `px-6 py-4 grid grid-cols-4 gap-4`. Four metric cards with trend indicators:
   - Revenue: ₱XX,XXX + ↑12% vs last period (green/red badge)
   - Patients: N + trend badge
   - Treatments: N + trend badge
   - Outstanding: ₱XX,XXX + trend badge
4. **Charts Row** — `px-6 py-4 grid grid-cols-2 gap-6`.
   - Left: Revenue Trend Line Chart (full period, collections vs outstanding)
   - Right: Top Treatments Bar Chart (top 10 by revenue, horizontal bars)
5. **Bottom Row** — `px-6 py-4 grid grid-cols-2 gap-6`.
   - Left: Payment Method Pie Chart (revenue by method)
   - Right: (reserved for future charts or empty in Phase 1)
6. **Solo Tier Upgrade** — if user is on Solo tier: Alert banner below header: "Analytics Dashboard is available on the Practice tier." + "Learn More" ghost button (opens Settings/billing) + "Maybe Later" ghost button (hides alert for session).

### Key Interactions

- **Time period toggle** → switching between Weekly / Monthly / Quarterly recalculates all metrics and re-renders all charts for the selected period. Default: Monthly.
- **Trend indicators** → compare current period vs previous period of same length. ↑ green if growth, ↓ red if decline. Percentage shown.
- **Chart hover/tap** → tooltips show exact values for the data point. Touch-friendly: tap to pin tooltip, tap elsewhere to dismiss.
- **"View Detailed Report"** → navigates to `/reports` with the current period mapped to a date range: Weekly → Custom (last 7 days from today), Monthly → This Month preset, Quarterly → Custom (last 90 days from today). The Reports screen opens with the mapped date range pre-applied.
- **Practice tier required** — FR11 is P1 (Practice tier and above). Solo tier: Alert banner replaces chart content with upgrade prompt. Same pattern as Prescriptions Modal upgrade prompt: "Learn More" + "Maybe Later."
- **Dashboard loads in <2s** — per FR11.1 acceptance criteria. Pre-compute aggregates if local dataset is large.
- **Offline** — fully functional offline. All charts render from local data.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Header + period toggle render. Summary cards show skeleton numbers. Charts show loading spinners. Must complete in <2s. |
| **Empty (new practice, no data)** | Summary cards show ₱0 / 0 / 0 / ₱0 with no trend badges. Charts show empty state: centered illustration + "Start seeing patients to track your practice performance." |
| **Solo tier** | Alert banner: "Analytics Dashboard is available on the Practice tier." + "Learn More" + "Maybe Later." **Charts and trend badges are hidden.** Summary metric cards still show real aggregate data (revenue, patients, treatments, outstanding) — these are not gated because the same data is available in Revenue Reports. Only the visual trend analysis (charts + trend %) is Practice-tier. |
| **Error** | Toast: "Failed to load dashboard" + Retry. |

### What This Screen Does

- Visualizes practice performance trends (revenue, patients, treatments, outstanding)
- Shows trend indicators vs previous period (↑/↓ %)
- Renders 3 chart types: revenue trend line, top treatments bar, payment method pie
- Supports weekly, monthly, and quarterly time periods
- Links to Revenue Reports for detailed drill-down
- Gates behind Practice tier (Solo tier sees upgrade prompt)

### What This Screen Does NOT Do

- Does NOT allow date-range customization — that is Revenue Reports
- Does NOT export data — use Revenue Reports for CSV/PDF export
- Does NOT show per-patient data — aggregate metrics only
- Does NOT forecast or predict — Phase 3 (FR23)
- Does NOT benchmark against external data — Phase 3 (FR23.2)
- Does NOT show per-dentist analytics — Phase 2 (FR16)
