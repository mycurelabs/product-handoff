---
screen: Dental Workspace
type: screen
route: /workspace/:patientId
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Select
  - DatePicker
  - Button
  - Separator
  - Table
  - TableHeader
  - TableRow
  - Checkbox
  - DropdownMenu
  - DropdownMenuItem
  - Badge
  - Avatar
  - Dialog
  - Sheet
  - AlertDialog
  - Sonner
wireframe: wireframes/dental-workspace.xml
wireframe_pages: [Default, Empty (New Patient), Past Baseline (Read-Only), Post-Payment]
  - Default
  - "Empty (New Patient)"
  - "Past Baseline (Read-Only)"
  - Post-Payment
---

## Screen: Dental Workspace

**Route:** `/workspace/:patientId`
**POV:** A dentist who has selected a patient from the Patient List and is now in their clinical session — charting, diagnosing, treating, and billing without ever leaving this screen.

### User Story

> As a dentist, I want to manage my entire patient session from a single workspace — viewing their dental history, diagnosing conditions, assigning treatments, and processing payment — so that I never need to navigate away during active patient care.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Continue to Payment | Primary | Opens Payment Modal with Grand Total (sum of Done rows only) | `default` |
| View All | Ghost | Opens modal showing full treatment history across all encounters | `ghost` |
| Mark Done (checkbox) | Secondary | Toggles treatment row from Proposed → Done; updates Grand Total | `outline` |
| ⋯ (overflow) | Ghost | Opens dropdown: Edit, Adjust Price, Remove | `ghost` |
| + (add session) | Secondary | Creates a new baseline session card in the carousel | `outline` |
| Prescriptions icon | Ghost | Opens Prescriptions Modal | `ghost` |
| Consent Forms icon | Ghost | Opens Consent Forms Modal | `ghost` |
| Attachments icon | Ghost | Opens Attachments Sheet | `ghost` |
| View toggle (↗) | Ghost | Toggles between full-mouth, quadrant, and full-view chart modes | `ghost` |
| Patient dropdown | Secondary | Switches patient context (reloads workspace) | `outline` |
| Date picker | Secondary | Changes baseline date; navigates carousel to that date's card or creates new | `outline` |
| Schedule Next Visit (post-payment) | Primary | Opens New Appointment modal as workspace overlay, patient pre-filled | `default` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Select` | Patient dropdown (top bar left) | `defaultValue={patientId}` |
| `DatePicker` | Baseline date picker (top bar center, "Today's Baseline - [date]") | `defaultValue={today}` |
| `Button` | Top bar action icons (Prescriptions, Consent Forms, Attachments, View toggle) | `variant='ghost'`, `size='icon'` |
| `Separator` | Vertical divider between document icons and view toggle | `orientation='vertical'` |
| `Table` | Breakdown table (Tooth, Surface, Condition, Treatment Plan, Work Done, Status, Total) | — |
| `TableHeader` | Column headers row | — |
| `TableRow` | Treatment rows | — |
| `Checkbox` | Work Done column per row | `onCheckedChange={markDone}` |
| `DropdownMenu` | Overflow menu (⋯) per row: Edit, Adjust Price, Remove | — |
| `DropdownMenuItem` | Edit action | — |
| `DropdownMenuItem` | Adjust Price action | — |
| `DropdownMenuItem` | Remove action (destructive) | `className='text-destructive'` |
| `Button` | "Continue to Payment" CTA (footer right) | `variant='default'` |
| `Button` | "View All" link (breakdown header right) | `variant='ghost'` |
| `Button` | "+ " add session button (carousel right) | `variant='outline'`, `size='icon'` |
| `Badge` | Treatment status: Pending (amber outline), Done (green outline), Paid (green filled) | `variant='outline'` for Pending/Done, `variant='default'` for Paid |
| `Avatar` | Patient photo in dropdown | `size='sm'` |
| `Dialog` | Payment Modal, Treatment History Modal, Prescriptions Modal, Consent Forms Modal containers | — |
| `Sheet` | Attachments Sheet (bottom) | `side='bottom'` |
| **Timeline Carousel** | Cover Flow + Dock magnification carousel (center zone) — CUSTOM COMPONENT, see epic | `baselines={Baseline[]}`, `activeIndex={number}`, `onCardFocus={(index) => void}`, `onToothTap={(toothId) => void}`, `viewMode={'full' \| 'quadrant' \| 'fullview'}` |
| **Interactive Dental Chart** | SVG dental diagram inside each carousel card — sub-component of Timeline Carousel (see epic Custom Components) | `teeth={Tooth[]}`, `dentition={'adult' \| 'pediatric'}`, `onToothTap={(toothId) => void}`, `viewMode={'full' \| 'quadrant'}`, `readOnly={boolean}` |
| `Button` | Carousel prev/next arrow buttons (flanking carousel) | `variant='outline'`, `size='icon'`, `min-h-[44px] min-w-[44px]` |
| `AlertDialog` | Discard guard (wizard abandonment, patient switch, close with unsaved data) | — |
| `AlertDialog` | Remove treatment confirmation | — |
| `Sonner` (Toast) | Work Done undo toast (5-second with Undo action) | `duration={5000}` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Patient | select (dropdown) | Yes | Must select a valid patient from local database | Current patient | `Session.patient_id` |
| Baseline Date | date (picker) | Yes | Valid date. Creates new baseline if none exists for selected date. | Today | `Baseline.date` |
| Adjust Price | number (inline edit) | Yes (when editing) | Must be ≥ 0. Numeric only. Currency format (₱). | Fee schedule default | `TreatmentRow.price` |
| Work Done | checkbox | No | Boolean toggle | Unchecked (Proposed) | `TreatmentRow.work_done` |
| Status | badge (auto) | — | Auto-derived: Proposed (unchecked), Done (checked, unpaid), Paid (checked, paid) | Proposed | `TreatmentRow.status` |

### Layout

1. **Top Bar** — `h-[56px] px-4 flex items-center justify-between border-b`. Left: patient avatar + Select dropdown. Center: "Today's Baseline - [Date]" DatePicker. Right: 4 icon Buttons (Prescriptions, Consent Forms, Attachments | Separator | View toggle).
2. **Carousel Zone** — `flex-1 relative overflow-hidden`. Contains Timeline Carousel (custom component). Cards arranged in 3D perspective with z-depth. Active card centered and largest. Past cards stack left with reduced scale/opacity. "+" button positioned at right edge of carousel.
3. **Breakdown Section** — `border-t`. Header row: left "Breakdown" text, right "View All" ghost button. Table with columns: Tooth, Surface, Condition, Treatment Plan, Work Done (Checkbox), Status, Total, ⋯ (DropdownMenu). Alternating row backgrounds. Grand Total row at bottom — **sums only Done (checked, unpaid) rows**, not all rows.
4. **Footer** — `sticky bottom-0 border-t bg-white px-6 py-3 flex justify-end`. "Continue to Payment" Button (amber/primary). Only visible when ≥1 treatment is marked Done.

### Key Interactions

- **Tap tooth on chart** → right Slideout opens with adaptive wizard (3 steps if no records, 4 steps if records exist). Workspace shrinks from 100% to ~70% width — slideout is NOT an overlay.
- **Swipe/drag carousel** → nearby cards scale up (Dock magnification), focal card enlarges with spring physics deceleration. Past cards show read-only historical chart state.
- **Tap carousel card (past baseline)** → breakdown table updates to show that baseline's treatments (read-only). The center label changes from **"Today's Baseline - [Date]"** to **"Baseline - [Date]"** (dropping "Today's") — a subtle but clear indicator that this is a historical view, not the active session. "Continue to Payment" CTA is hidden. Work Done checkboxes are disabled. Overflow menus are hidden. Active baseline restores on swipe back to front (badge disappears, CTAs re-enable).
- **Tapping a tooth on a past baseline chart** → opens the Slideout at Overview step (read-only) showing that tooth's records at that date. The wizard does NOT open for new conditions. A "Work on this tooth" button at the bottom of the Overview redirects the dentist to the current/active baseline and opens the wizard for that tooth there. If no active baseline exists for today, prompts: "Create a new baseline for today?"
- **Check "Work Done" checkbox** → treatment status changes from Proposed to Done. Grand Total recalculates (sums Done rows only — Proposed and Paid rows are excluded). "Continue to Payment" CTA appears when Grand Total > 0.
- **Tap ⋯ overflow on row** → DropdownMenu: Edit (opens Slideout at Review step), Adjust Price (inline edit), Remove (confirmation dialog).
- **Tap treatment row body** (not checkbox/overflow) → opens Slideout to review/edit that treatment.
- **Tap "View All"** → opens Treatment History Modal showing ALL treatments from ALL encounters for this patient. Same modal regardless of which baseline is focused in the carousel. This is the single comprehensive record view.
- **Tap "Continue to Payment"** → opens Payment Modal with completed treatments total.
- **Date picker change** → carousel navigates to that date's baseline card. If no baseline exists for that date, creates a new active session card.
- **Patient dropdown change** → workspace reloads for the selected patient.
- **Top bar icons** → open respective overlays (Prescriptions Modal, Consent Forms Modal, Attachments Sheet).
- **View toggle** → cycles between full-mouth (default), quadrant (one quadrant at a time with nav), and full-view (chart takes over viewport with exit button). **Tapping a tooth in quadrant view** still opens the slideout wizard — the dentist stays in quadrant view while working in the wizard. The quadrant context is preserved. In full-view mode, tapping a tooth also opens the wizard; full-view shrinks to accommodate the slideout.
- **Pediatric patient** → carousel chart switches to deciduous dentition (20 teeth, FDI 51-85).
- **PWD/Senior patient** → discount auto-applied in Payment Modal.
- **Proposed treatments not marked Done** → carry forward to next baseline (persistent until acted on or removed).
- **Surface state colors on chart:** RED (#DC2626) = condition diagnosed, BLUE (#2563EB) = treatment assigned. Colors update in real-time as wizard progresses.
- **Missing/extracted teeth** → teeth marked as missing, extracted, or congenitally absent render as a distinct visual state on the chart (grayed out, "X" overlay). Tapping a missing tooth shows a lightweight status card ("Tooth 16 — Extracted, 2024-01-15") instead of opening the full wizard. Status can be set via the tooth's Overview step or a quick-action on the chart.
- **Re-entering a charted tooth** → if a tooth already has treatments in the current breakdown table, tapping it opens the Slideout at Overview (4-step flow) showing existing records. The dentist can add MORE conditions via "Create Record" or edit existing ones. The wizard does NOT open fresh — it always shows existing context first.
- **Work Done undo** → checking "Work Done" triggers a 5-second undo toast: "Marked as done. [Undo]". Tapping "Undo" restores the row to Proposed state. After 5 seconds, the toast dismisses and the action is committed. The undo toast uses a large tap target (≥44px) for gloves-on use. **If the dentist takes any other action while the toast is active** (opens slideout, taps another row, changes carousel date), the pending action commits immediately and the toast dismisses — navigation is never blocked by an undo window.
- **Close workspace guard** → if the slideout wizard is open with unsaved data (any condition/treatment selections in progress), the close/back button triggers an AlertDialog: "You have unsaved work on Tooth [N]. Discard?" with large Cancel + Discard buttons. If no wizard is open, close is immediate — no confirmation.
- **Patient dropdown guard** → switching patients mid-session triggers the same discard guard as close. If wizard has unsaved data: "Switching patients will discard unsaved work on Tooth [N]. Continue?" If no unsaved data: patient switch is immediate.
- **Date picker behavior** → changing the date navigates the carousel to that date's baseline card. The breakdown table follows the active carousel card, showing treatments for the focused baseline. Existing session data is never destroyed — it remains on its baseline card and can be reached by swiping back. If no baseline exists for the selected date, a new empty baseline card is created. **Date picker is subject to the same discard guard as close/patient-switch** — if the slideout wizard has unsaved data, an AlertDialog fires before the carousel navigates.
- **Carousel arrow buttons** → visible prev/next arrow buttons (≥44px each) flank the carousel for non-gesture navigation. Required for WCAG 2.5.7 (drag alternative) and gloves-on reliability. Additionally, a tooth-number quick-jump (row of numbered indicators or dropdown) allows jumping directly to a specific tooth.
- **Finalize → auto-scroll + highlight** → after "Finalize Plans" closes the slideout, the breakdown table auto-scrolls to reveal the newly added rows. New rows have a brief highlight animation (amber flash, fades after 3 seconds). Focus returns to the carousel chart so the dentist can immediately tap the next tooth.

> **Clinical Color Standard:** RED (#DC2626) = conditions/problems, BLUE (#2563EB) = treatments/completed work. This follows the universal dental charting convention (see `dentalchart/DENTAL_CHART_REFERENCE.md`). Brand amber (#FFCC5E) is used for UI elements only, never for clinical surface state.
- **Breakdown row ordering** → rows are ordered by tooth number (FDI ascending: 11, 12, ... 48), then by creation time within the same tooth. This ensures a predictable reading order for billing review.
- **iPad sleep / session persistence** → wizard state (current step, selected surfaces, assigned conditions, treatments, notes) is persisted to local storage on every step advancement. If the iPad auto-locks and the dentist returns, the wizard resumes at the exact step with all selections preserved.
- **Touch targets** → all interactive elements meet ≥44x44px minimum: Work Done checkbox (padded hit area), overflow ⋯ icon (padded hit area), carousel arrow buttons, surface selector regions, wizard step indicators. This is a hard requirement for iPad conversion and gloves-on clinical use.
- **Schedule Next Visit (FR6.4)** → after payment is completed (Payment Modal "Complete" tapped), a follow-up prompt appears: "Schedule next visit?" with "Schedule" (primary) + "Not now" (ghost). Tapping "Schedule" opens the New Appointment modal (same as Module 3) as an overlay within the workspace, pre-filled with the current patient. After appointment creation: toast "Next visit scheduled for [date] at [time]." An **"Appointment Set" indicator** (subtle badge or icon near the top bar) persists for the session so the dentist knows a follow-up was booked. "Not now" dismisses the prompt — no appointment created.
- **Treatment row lifecycle (Proposed → Done → Paid):**
  - **Proposed** — unchecked checkbox, "Pending" badge (amber outline), row fully interactive (overflow menu: Edit, Adjust Price, Remove). NOT included in Grand Total.
  - **Done** — checked checkbox, "Done" badge (green outline), row fully interactive. Included in Grand Total. Eligible for payment.
  - **Paid** — checkbox locked/disabled (checked but grayed), "Paid" badge (green filled), row text muted (#4A6B7A), overflow menu hidden. NOT included in Grand Total (already billed). Cannot be edited, price-adjusted, or removed.
  - **Transition: Done → Paid** — triggered automatically when Payment Modal "Complete" is tapped. All Done rows included in the invoice change to Paid. Grand Total recalculates to exclude newly-paid rows. If Grand Total reaches ₱0, "Continue to Payment" CTA disappears.
  - **Post-payment continuation** — after payment, the dentist can still add new treatments in the same session (wizard still works). New rows appear as Proposed. Paid rows remain visible in the breakdown table for the current session as a billing receipt. On next session/baseline, Paid rows from previous sessions are only visible in the Treatment History Modal.
  - **Partial payment** — if the dentist pays for some Done rows but not all (via Payment Modal partial payment), only the covered rows transition to Paid. Remaining Done rows stay as Done with Grand Total reflecting the unpaid balance.
- **FR1.7 intentional deviation** → the PRD specifies dual footer totals (Estimated left + Checkout right). The hi-fi mockup simplifies this to a single "Continue to Payment" CTA. **Mockup is authoritative.** The Grand Total row in the breakdown table serves the "estimated" purpose. The CTA serves the "checkout" purpose. Dual-footer is not implemented.

### Carousel Card Render Strategy

Each carousel card renders a **simplified per-quadrant dental chart** — not the full interactive 32-tooth SVG.

| Aspect | Spec |
|--------|------|
| Render mode | Pre-rendered simplified diagram per card (not live interactive SVG) |
| Chart view | Per-quadrant view within each card — upper-left, upper-right, lower-left, lower-right |
| Interaction | Tap on a tooth area opens the slideout wizard with that tooth pre-selected |
| Full interactive chart | Only rendered when the slideout opens or in full-view mode — not inside carousel cards |
| Performance rationale | Full live SVG across 3-5 visible carousel cards would require 480-800+ SVG elements in view simultaneously, causing scroll jank on iPad |
| Re-render trigger | Card snapshot re-generates when treatment data changes (condition added, treatment assigned, work done) |

> **Why not full live SVG in cards?** The carousel has 3-5 cards visible at once with Cover Flow perspective transforms. Each full chart has 32 teeth × 5 surfaces = 160 SVG interaction zones. Multiplied across visible cards, this exceeds iPad GPU budgets for smooth 60fps gesture animation. The simplified per-quadrant render preserves the visual signature while keeping the carousel buttery smooth.

### Dental Chart Layout

The interactive dental chart is a **16×2 grid** (16 columns, 2 rows — upper arch and lower arch).

| Aspect | Spec |
|--------|------|
| Grid | 16 columns × 2 rows |
| Upper row | Teeth 1-16 (right to left: 3rd molar → central incisor, then central incisor → 3rd molar) |
| Lower row | Teeth 32-17 (right to left: mirroring upper arch) |
| Per-tooth SVG | Individual SVG file per tooth, each with pre-defined surface zones (Buccal, Mesial, Distal, Incisal/Occlusal, Palatal/Lingual + Cervical) |
| Tooth cell size | Fixed frame per tooth in the grid — teeth are not variable-width |
| Numbering | Universal Numbering System (1-32) for adult, with FDI (11-48) as alternate display option |
| Pediatric | 20 teeth (FDI 51-85) — same grid structure with 12 positions empty/hidden |
| Color standard | RED (#DC2626) = conditions, BLUE (#2563EB) = treatments (see Clinical Color Standard note) |
| Surface zones | Each tooth SVG has colorable surface regions ready for per-surface state fill |

> The dev team already has individual tooth SVGs with surface zones pre-built. The grid arranges them in the standard dental chart layout (upper arch top, lower arch bottom, patient's right on viewer's left).

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Back/Close | Patient List | `/patients` | Module 2: Patient Management |
| Patient dropdown (profile link) | Patient Profile | `/patients/:id` | Module 2: Patient Management |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Top bar renders with patient name. Carousel shows skeleton card placeholder. Breakdown table shows shimmer rows. |
| **Empty (new patient, no baselines)** | Carousel shows single empty card with tooth icon + "No baselines yet. Tap a tooth to start." Breakdown table empty state: "No treatments added." No "Continue to Payment" CTA. |
| **Empty (baseline exists, no treatments)** | Carousel shows chart. Breakdown table: "No treatments in this session. Tap a tooth on the chart to add." |
| **Post-payment (all Done paid)** | All previously-Done rows show "Paid" badge (green filled), locked checkboxes, muted text, no overflow menus. Grand Total: ₱0. "Continue to Payment" CTA hidden. Carousel and wizard still functional for new treatments. |
| **Post-payment (partial)** | Mix of Paid rows (locked, muted) and remaining Done/Proposed rows (interactive). Grand Total reflects only unpaid Done rows. "Continue to Payment" CTA visible if Done rows remain. |
| **Error** | Toast notification "Failed to load patient data" + Retry button. Workspace remains on last good state if available. |

### CRUD Interactions

#### Adding (treatments via wizard)
- **Trigger:** Tap tooth on chart OR tap "+ " button in carousel
- **Opens:** Right slideout panel (resizes workspace, not overlay). Adaptive wizard: 3 steps (no records) or 4 steps (with records).
- **Form fields:** See Slideout step comments for per-step field specs
- **Validation:** At least one surface-condition pair required before advancing past Condition step. At least one treatment per condition required before advancing past Treatment step.
- **On success:** "Finalize Plans" → rows added to breakdown table with auto-filled prices from fee schedule. Slideout closes. Workspace returns to full width.
- **On cancel:** Back button or ✕ → discard wizard state, slideout closes.

#### Editing (treatment row)
- **Trigger:** Overflow menu → Edit, OR tap row body
- **Opens:** Slideout at Review step (Step 3/4) with existing data pre-filled
- **Differences from Create:** Review step shows existing condition-treatment pairs. Editable. "Back" navigates to prior steps.
- **On success:** Row updated in breakdown table.

#### Adjusting Price
- **Trigger:** Overflow menu → Adjust Price
- **Opens:** Inline price edit on the row (Total column becomes editable)
- **Validation:** Must be ≥ 0. Numeric only.
- **Blocked when:** Row status is Paid (overflow menu hidden on Paid rows).
- **On success:** Price updates. Grand Total recalculates (Done rows only).

#### Removing
- **Trigger:** Overflow menu → Remove
- **Confirmation:** AlertDialog: "Remove this treatment? This action cannot be undone." with Cancel + Remove (destructive) buttons.
- **Soft vs hard delete:** Hard delete from current session. Does not affect historical baselines.
- **Blocked when:** Row status is Paid (overflow menu hidden on Paid rows). Paid treatments cannot be removed.
- **On success:** Row removed. Grand Total recalculates (Done rows only).

### What This Screen Does

- Renders the complete dental workspace with timeline carousel, interactive chart, and treatment breakdown
- Manages the full treatment lifecycle: Proposed → Done → Paid
- Provides Cover Flow + Dock magnification navigation through patient visit history
- Hosts all clinical overlays (slideout wizard, payment modal, prescriptions, consent forms, attachments)
- Supports adult (32 teeth) and pediatric (20 teeth) dentitions
- Auto-fills treatment prices from configurable fee schedule
- Tracks work done status per treatment row
- Generates invoices and processes payments via Payment Modal

### What This Screen Does NOT Do

- Does NOT manage patient registration or profile editing — that is Module 2: Patient Management
- Does NOT manage the full calendar or appointment CRUD — that is Module 3: Scheduling. The workspace only supports a "Schedule Next Visit" quick-action (see Key Interactions) that opens the New Appointment modal as an overlay within the workspace.
- Does NOT generate revenue reports — that is Module 4: Reporting & Analytics
- Does NOT manage staff permissions — that is Module 5: Staff Management
- Does NOT process online payments (GCash/Maya) — Phase 2. "Generate Payment Link" is placeholder.
- Does NOT support multi-dentist workspace — Phase 2 (FR16)
- Does NOT render on mobile viewports — tablet only (≥ 1024px)

### Wireframe

See [wireframes/dental-workspace.xml](wireframes/dental-workspace.xml)
