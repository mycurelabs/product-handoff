---
screen: "Slideout — Condition"
type: step
route: null
module: "Module 1: Dental Workspace"
tier: 1
components: [Custom Stepper, 5-Surface Selector, Switch, Select, SelectGroup, SelectItem, Card, Button, Badge]
wireframe: wireframes/step-slideout-condition.xml
wireframe_pages: [Default, Empty]
---

## Step: Slideout — Condition (of Dental Workspace)

**Route:** N/A (panel within `/workspace/:patientId`)
**POV:** A dentist selecting which tooth surfaces are affected and assigning dental conditions — the diagnostic step of the wizard.

### User Story

> As a dentist, I want to select specific tooth surfaces and assign conditions (Caries, Fractured, Missing, etc.) so that each surface-condition pair is documented and carried forward to treatment selection.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Save & Continue | Primary | Saves condition assignments, advances to Treatment step | `default` |
| Back | Ghost | Returns to Overview step (if records exist) or closes slideout (if no records) | `ghost` |
| ✕ | Ghost | Closes slideout, discards wizard state | `ghost` |
| 🗑 (trash) | Ghost | Removes a condition card from the stack | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| **Custom Stepper** | Progress indicator. If 3-step flow: Step 1 active (Condition). If 4-step: Step 2 active. | `activeStep={currentStep}` |
| **5-Surface Selector** | Interactive BMDIP diagram with cervical mode toggle. Multi-select surfaces (RED #DC2626 highlight). Label updates: "Surface M selected" / "Surfaces M, D selected" (CUSTOM COMPONENT) | `readOnly={false}`, `multiSelect={true}` |
| `Switch` | Cervical mode toggle — enables cervical zone selection on the tooth diagram alongside standard surfaces | `label='Cervical'` |
| `Select` | Conditions dropdown with grouped options | — |
| `SelectGroup` | "WHOLE TOOTH" group header (Impacted, Supernumerary, Missing, Malpositioned) | `label='WHOLE TOOTH'` |
| `SelectGroup` | "SURFACE" group header (Caries, Decayed, Fractured, etc.) | `label='SURFACE'` |
| `SelectItem` | Individual condition options with dental icons | — |
| `Card` | Condition card (read-only summary of assigned surface-condition pair) | — |
| `Button` | Trash icon on each condition card | `variant='ghost'`, `size='icon'` |
| `Badge` | Counter: "Conditions ([N])" showing total surfaces affected | — |
| `Button` | "Save & Continue" footer CTA | `variant='default'` |
| `Button` | "Back" footer CTA | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Surface selection | multi-select (BMDIP toggles) | Yes (≥1) | At least one surface or cervical zone must be selected before condition dropdown is enabled | None | `TreatmentRecord.surfaces` |
| Cervical mode | toggle (Switch) | No | When ON, cervical zones on the tooth diagram become selectable alongside standard surfaces | OFF | `TreatmentRecord.cervical_enabled` |
| Condition | select (grouped dropdown) | Yes | Must select from structured list — no free-text | None | `TreatmentRecord.condition` |

### Layout

1. **Header** — `px-4 py-3 flex justify-between`. "Tooth #[N]" (bold) + ✕ close.
2. **Stepper** — `px-4 py-2`. Numbered circles. Active step highlighted amber.
3. **Surface Diagram** — `px-4 py-4 flex justify-center`. Interactive 5-surface selector with cervical toggle (Switch) positioned above or beside the diagram. When cervical mode is ON, cervical zones appear on the tooth neck area as additional tappable regions. Selected surfaces fill RED (#DC2626). Label below: "Surface [X] selected" or "Surfaces M, D, Cervical selected".
4. **Conditions Section** — `px-4 py-2`. Header: "Conditions ([N])" with count. Select dropdown: "Select condition" placeholder. Grouped options (WHOLE TOOTH / SURFACE). Below dropdown: stacked condition cards showing assigned pairs (e.g., "M, D · Caries" + trash icon).
5. **Empty State** — When no conditions assigned: tooth icon + "No conditions added" + "You have not added any conditions yet." centered below dropdown.
6. **Footer** — `sticky bottom-0 px-4 py-3 border-t flex justify-between gap-3`. "Back" ghost button (left). "Save & Continue" primary button (right).

### Key Interactions

- **Tap surface on diagram** → surface highlights RED (#DC2626). Label updates. Multiple surfaces can be selected simultaneously.
- **Tap selected surface again** → deselects it.
- **Select condition from dropdown** → condition card appears below showing the surface-condition pair (e.g., "M, D · Caries"). Dropdown resets to placeholder for next assignment.
- **Condition count "([N])"** → tracks total number of surfaces affected across all cards (not number of cards).
- **Tap trash on condition card** → removes that surface-condition pair. Count decrements.
- **Multiple assignments** → select different surfaces, pick another condition → new card stacks below existing ones. Same tooth can have multiple surface-condition pairs.
- **"Save & Continue"** → validates ≥1 condition card exists. Advances to Treatment step.
- **"Back"** → returns to Overview step (4-step flow) or closes slideout (3-step flow). Discards unsaved condition state.
- **Whole-tooth conditions** (Impacted, Missing, etc.) → automatically select all surfaces. Surface diagram shows full tooth highlighted.
- **Cervical toggle ON** → cervical zones on the tooth diagram become tappable (gum-line region). Standard BMDIP surfaces remain selectable simultaneously. Cervical selections appear in condition cards as "Cervical" alongside surface labels (e.g., "M, Cervical · Caries"). Toggle state persists within the wizard session. Common use: cervical caries, abrasion, gingival recession.
- **Cervical toggle OFF (default)** → cervical zones are not shown on the diagram. Only standard 5 surfaces (B, M, D, I, P) are selectable.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — step loads instantly from wizard state. |
| **Empty (no conditions yet)** | Surface diagram interactive but no surfaces selected. Dropdown enabled. Empty state illustration: tooth icon + "No conditions added" message. |
| **Error** | N/A — local-only step, no async operations. |

### CRUD Interactions

#### Adding (condition assignment)
- **Trigger:** Select surfaces on diagram + pick condition from grouped dropdown
- **Opens:** Inline — condition card appears below dropdown (no separate modal)
- **Validation:** At least one surface must be selected before condition dropdown is enabled. Condition must be selected from structured list.
- **On success:** Condition card appears showing surface-condition pair (e.g., "M, D · Caries") with trash icon. Count updates.
- **On cancel:** N/A — cards appear immediately on selection.

#### Deleting (condition card)
- **Trigger:** Trash icon on condition card
- **Confirmation:** None — immediate removal (low-risk: wizard state only, not persisted to database yet)
- **On success:** Card removed from stack. Count decrements. Surfaces deselected if no longer referenced.

### What This Panel Does

- Provides interactive surface selection via the 5-surface BMDIP diagram with cervical mode toggle
- Offers structured condition assignment via grouped dropdown (WHOLE TOOTH / SURFACE categories)
- Stacks multiple surface-condition pairs as deletable cards
- Tracks total affected surfaces count
- Validates that at least one condition is assigned before advancing

### What This Panel Does NOT Do

- Does NOT assign treatments — that is the Treatment step
- Does NOT show pricing — pricing appears only in the breakdown table after finalization
- Does NOT save to the database — data is held in wizard state until "Finalize Plans" on the Review step

### Wireframe

See [`wireframes/step-slideout-condition.xml`](wireframes/step-slideout-condition.xml) — 2 pages: Default, Empty.
