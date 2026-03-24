---
screen: "Slideout — Treatment"
type: step
route: null
module: "Module 1: Dental Workspace"
tier: 1
components: [Custom Stepper, 5-Surface Selector, Select, SelectGroup, SelectItem, Button]
wireframe: wireframes/step-slideout-treatment.xml
wireframe_pages: [Default]
---

## Step: Slideout — Treatment (of Dental Workspace)

**Route:** N/A (panel within `/workspace/:patientId`)
**POV:** A dentist who has diagnosed conditions and is now selecting the appropriate treatment for each condition, guided by dental specialty categories.

### User Story

> As a dentist, I want to select a treatment for each diagnosed condition — filtered by dental specialty — so that each condition-treatment pair is documented before review.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Save & Continue | Primary | Saves treatment assignments, advances to Review step | `default` |
| Back | Ghost | Returns to Condition step | `ghost` |
| ✕ | Ghost | Closes slideout, discards wizard state | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| **Custom Stepper** | Progress indicator. Treatment step active (amber). | `activeStep={currentStep}` |
| **5-Surface Selector** | Read-only diagram showing current state. Surfaces change from RED (#DC2626) (diagnosed) to BLUE (#2563EB) (treatment assigned) as treatments are selected. | `readOnly={true}` |
| `Select` | Treatment dropdown per condition card. Grouped by dental specialty. | — |
| `SelectGroup` | Specialty group headers: ORTHO, ENDO, PERIO, PROSTHO, ORAL, PEDIA, COSMETIC | `label='[SPECIALTY]'` |
| `SelectItem` | Individual treatment options with dental icons | — |
| `Button` | "Save & Continue" footer CTA | `variant='default'` |
| `Button` | "Back" footer CTA | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Treatment | select (grouped by specialty) | Yes (per condition) | Each condition card must have a treatment selected before advancing | None | `TreatmentRecord.treatment` |

### Layout

1. **Header** — `px-4 py-3 flex justify-between`. "Tooth #[N]" + ✕.
2. **Stepper** — Treatment step active (amber).
3. **Surface Diagram** — Read-only. Surfaces update color as treatments are assigned: RED (#DC2626) → BLUE (#2563EB).
4. **Treatment Plans Section** — `px-4 py-2 space-y-4`. Header: "Treatment Plans". For each condition card from the Condition step:
   - Context row: surface labels (left, e.g., "M, D") + condition badge (right, e.g., "Caries")
   - Treatment Select dropdown below: "Select a treatment" placeholder. Grouped options by specialty.
   - When selected: dropdown shows selected treatment with dental icon (e.g., "Filling").
5. **Footer** — `sticky bottom-0 px-4 py-3 border-t flex justify-between`. "Back" + "Save & Continue".

### Key Interactions

- **Each condition card from the Condition step** gets its own treatment dropdown. The condition context (surfaces + condition name) is shown as a label above each dropdown.
- **Select treatment** → dropdown updates to show selected treatment. Surface diagram updates: affected surfaces change from RED (#DC2626) to BLUE (#2563EB).
- **All treatments must be assigned** before "Save & Continue" is enabled. Button is disabled if any condition card has no treatment selected.
- **Treatment dropdown grouped by specialty** — ORTHO (Filling, Cleaning), ENDO (Root Canal, Pulpectomy), PERIO (Scaling, Debridement), PROSTHO (Crown, Bridge, Denture), ORAL (Extraction), PEDIA (Sealant, Fluoride), COSMETIC (Bleaching, Veneer). List is configurable by the dentist in Settings.
- **"Back"** → returns to Condition step. Treatment selections are preserved if returning forward.
- **Treatment may auto-qualify** — e.g., selecting "Filling" for Caries may display as "Composite Filling" in the Review step (treatment specificity added based on condition context).

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — loads from wizard state. |
| **Empty** | N/A — this step always has at least one condition card (validated in prior step). |
| **Error** | N/A — local-only step, no async operations. |

### CRUD Interactions

#### Adding (treatment assignment)
- **Trigger:** Select a treatment from the specialty-grouped dropdown for each condition card
- **Opens:** Inline — dropdown updates to show selected treatment (no separate modal)
- **Validation:** All condition cards must have a treatment selected before "Save & Continue" is enabled.
- **On success:** Treatment shown in dropdown. Surface diagram updates (RED #DC2626 → BLUE #2563EB). "Save & Continue" becomes enabled when all conditions have treatments.
- **On cancel:** N/A — selecting a different treatment replaces the previous selection inline.

### What This Panel Does

- Presents each diagnosed condition with a treatment selection dropdown
- Groups treatments by dental specialty for guided selection
- Updates the surface diagram in real-time (RED #DC2626 → BLUE #2563EB as treatments are assigned)
- Validates all conditions have treatments before advancing

### What This Panel Does NOT Do

- Does NOT show pricing — prices auto-fill in the breakdown table after finalization
- Does NOT allow adding new conditions — go Back to the Condition step for that
- Does NOT save to database — held in wizard state until Review step finalization

### Wireframe

See [`wireframes/step-slideout-treatment.xml`](wireframes/step-slideout-treatment.xml) — 1 page: Default.
