---
screen: "Slideout — Review"
type: step
route: "N/A (panel within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Card
  - Badge
  - Textarea
  - Button
wireframe: wireframes/step-slideout-review.xml
wireframe_pages: [Default]
  - Default
---

## Step: Slideout — Review (of Dental Workspace)

**Route:** N/A (panel within `/workspace/:patientId`)
**POV:** A dentist reviewing the complete summary of diagnosed conditions and assigned treatments before committing them to the breakdown table.

### User Story

> As a dentist, I want to review all condition-treatment pairs with optional notes before finalizing, so that I can confirm accuracy and add clinical notes per treatment.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Finalize Plans | Primary | Commits all condition-treatment pairs to the breakdown table with auto-filled prices. Closes slideout. | `default` |
| Back | Ghost | Returns to Treatment step | `ghost` |
| ✕ | Ghost | Closes slideout, discards wizard state | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| **Custom Stepper** | Progress indicator. All steps complete (amber). Review step active. | `activeStep={lastStep}` |
| **5-Surface Selector** | Read-only diagram. Final state: treated surfaces in blue. | `readOnly={true}` |
| `Card` | Summary card per condition-treatment pair. Shows: surfaces, condition badge, treatment badge. | — |
| `Badge` | Condition badge (e.g., "🦷 Caries") | `variant='outline'` |
| `Badge` | Treatment badge (e.g., "🦷 Composite Filling") | `variant='outline'` |
| `Textarea` | "Record Note" free-text field per summary card | `placeholder='Type your message here.'` |
| `Button` | "Finalize Plans" footer CTA | `variant='default'` |
| `Button` | "Back" footer CTA | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Record Note | textarea | No | Max 500 characters | Empty | `TreatmentRecord.note` |

### Layout

1. **Header** — `px-4 py-3 flex justify-between`. "Tooth #[N]" + ✕.
2. **Stepper** — All steps completed (amber circles). Review step is the active/final step.
3. **Surface Diagram** — Read-only. Final state with treated surfaces in blue.
4. **Summary of Plans** — `px-4 py-2 space-y-4`. Header: "Summary of Plans". Scrollable card stack:
   - Each card: surface labels (bold, e.g., "M, D") + condition (🦷 Caries) + treatment (🦷 Composite Filling). Below: "Record Note" label + Textarea ("Type your message here."). One card per condition-treatment pair.
5. **Footer** — `sticky bottom-0 px-4 py-3 border-t flex justify-between`. "Back" ghost (left). "Finalize Plans" primary (right).

### Key Interactions

- **Summary cards are read-only for condition/treatment** — to change assignments, use "Back" to return to prior steps.
- **Record Note is optional** — dentist can add per-treatment clinical notes. Each card has its own note field.
- **"Finalize Plans"** → writes all records to the patient's baseline. For each condition-treatment pair, a new row appears in the breakdown table with:
  - Tooth number, surfaces, condition, treatment, auto-filled price from fee schedule, Work Done unchecked, status empty.
- **Slideout closes** after finalization. Workspace returns to full width. Breakdown table and Grand Total update.
- **"Back"** → returns to Treatment step. Notes are preserved.
- **"✕"** → closes slideout. All wizard state discarded (conditions, treatments, notes — nothing saved).

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — loads from wizard state. |
| **Empty** | N/A — always has at least one summary card. |
| **Error** | If save fails: Toast "Failed to save treatment record" + Retry. Slideout stays open. |

### CRUD Interactions

#### Creating (finalize treatment records)
- **Trigger:** "Finalize Plans" button
- **Opens:** N/A — commits directly, no additional modal
- **Form fields:** Summary cards are read-only. Only "Record Note" textarea per card is editable.
- **Validation:** At least one summary card must exist (always true at this step). Record Note is optional.
- **On success:** For each condition-treatment pair: a new row is created in the breakdown table with auto-filled price from fee schedule. Slideout closes. Workspace returns to full width. Table auto-scrolls to new rows with amber highlight. Focus returns to carousel.
- **On cancel:** "Back" returns to Treatment step. "✕" discards all wizard state — no records created.
- **On error:** Toast "Failed to save treatment record" + Retry. Slideout stays open.

### What This Panel Does

- Shows a complete summary of all condition-treatment pairs before committing
- Provides per-treatment "Record Note" fields for clinical annotations
- Commits records to the breakdown table with auto-filled pricing on "Finalize Plans"
- Closes the slideout and restores workspace to full width

### What This Panel Does NOT Do

- Does NOT allow editing conditions or treatments directly — use "Back" to navigate to prior steps
- Does NOT show pricing — prices appear only in the breakdown table after finalization
- Does NOT mark treatments as "Done" — that happens in the breakdown table via the Work Done checkbox

### Wireframe

See [wireframes/step-slideout-review.xml](wireframes/step-slideout-review.xml)
