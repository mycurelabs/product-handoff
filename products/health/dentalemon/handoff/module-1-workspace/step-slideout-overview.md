---
screen: "Slideout — Overview"
type: step
route: "N/A (panel within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Sheet
  - Badge
  - Button
wireframe: wireframes/step-slideout-overview.xml
wireframe_pages: [Default, Past Baseline]
  - Default
  - "Past Baseline"
---

## Step: Slideout — Overview (of Dental Workspace)

**Route:** N/A (panel within `/workspace/:patientId`)
**POV:** A dentist who tapped a tooth that has existing treatment records, arriving at the overview to review history before creating a new record.

### User Story

> As a dentist, I want to see a tooth's recent treatment history at a glance so that I can make informed decisions about new conditions and treatments.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Create Record | Primary | Advances to Condition step (Step 2) | `default` |
| View All | Ghost | Opens full treatment history for this tooth across all encounters | `ghost` |
| ✕ | Ghost | Closes slideout, returns workspace to full width | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Sheet` | Slideout container (right side, 320px, resizes workspace) | `side='right'` |
| **Custom Stepper** | 4-step progress indicator: Overview (active) → Condition → Treatment → Review | `activeStep={0}` |
| **5-Surface Selector** | Read-only tooth surface diagram showing existing conditions (CUSTOM COMPONENT) | `readOnly={true}` |
| `Card` | Recent record entries (surface labels + condition + treatment badges + "Done" badge) | — |
| `Badge` | Condition badge (e.g., "Caries"), Treatment badge (e.g., "Composite Filling"), status badge ("Done") | `variant='outline'` |
| `Button` | "Create Record" CTA | `variant='default'` |
| `Button` | "View All" link | `variant='ghost'` |
| `Button` | Close (✕) | `variant='ghost'`, `size='icon'` |

### Layout

1. **Header** — `px-4 py-3 flex justify-between items-center`. Left: "Tooth #[N]" (bold). Right: ✕ close button.
2. **Stepper** — `px-4 py-2`. 4 numbered circles with connecting lines. Step 1 (Overview) active/amber. Steps 2-4 inactive/gray. Labels: Overview, Condition, Treatment, Review.
3. **Surface Diagram** — `px-4 py-4 flex justify-center`. Read-only 5-surface selector showing existing condition state. Surfaces colored by current status.
4. **Recent Records** — `px-4 py-2 space-y-3`. Header: "Recent Records". Cards showing: surface labels (e.g., "M, D") + "Done" badge + condition (🦷 Caries) + treatment (🦷 Composite Filling). Each card is a past record entry.
5. **Footer** — `sticky bottom-0 px-4 py-3 border-t flex justify-between`. "View All" ghost link (left). "Create Record" primary button (right).

### Key Interactions

- **Step is conditional** — this step ONLY appears when the tooth has existing records. If the tooth has no records, the wizard starts at the Condition step (3-step flow).
- **Tap "Create Record"** → advances to Condition step. Surface diagram becomes interactive.
- **Tap "View All"** → opens the Treatment History Modal filtered to this tooth only, showing all records for this tooth across all encounters. The modal's date-grouped layout remains the same but entries are scoped to the selected tooth number.
- **Recent records are read-only** — tapping a record card does nothing. These are context, not actions.
- **Surface diagram is read-only** on this step — shows existing state but does not accept input.
- **Past baseline context** — when opened from a past baseline (not today's), the "Create Record" button is replaced by a **"Work on this tooth"** button. On tap: (1) slideout closes, (2) carousel navigates to today's active baseline, (3) slideout reopens for this tooth on the active baseline (Overview step if records exist, Condition step if not). If no active baseline exists for today, an inline prompt appears first: "Create a new baseline for today?" with Create + Cancel. Create → new baseline created → slideout opens. Cancel → stays on past baseline.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Stepper renders. Surface diagram placeholder. Shimmer cards in Recent Records zone. |
| **Empty** | N/A — this step is never shown for teeth without records. |
| **Error** | Toast: "Failed to load tooth history" + Retry. Surface diagram and records fallback to empty. |

### What This Panel Does

- Shows the tapped tooth's identity (number, name)
- Displays a read-only surface diagram with current condition state
- Lists recent treatment records from previous encounters with condition + treatment badges
- Provides "Create Record" entry point to start a new diagnosis
- Provides "View All" to see complete tooth history

### What This Panel Does NOT Do

- Does NOT allow editing existing records — those are historical and read-only
- Does NOT accept surface selection input — surface selector becomes interactive only on the Condition step
- Does NOT show for teeth with no records — wizard skips directly to Condition step

### Wireframe

See [wireframes/step-slideout-overview.xml](wireframes/step-slideout-overview.xml)
