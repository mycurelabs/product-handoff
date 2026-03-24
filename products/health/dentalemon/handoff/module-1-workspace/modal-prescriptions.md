---
screen: Prescriptions
type: modal
route: "N/A (modal within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Dialog
  - DialogHeader
  - Textarea
  - Input
  - Button
wireframe: wireframes/modal-prescriptions.xml
wireframe_pages: [Default, Solo Tier Upgrade]
  - Default
  - "Solo Tier Upgrade"
---

## Modal: Prescriptions (of Dental Workspace)

**Route:** N/A (modal within `/workspace/:patientId`)
**POV:** A dentist generating a prescription during a patient session, with PRC credentials and clinic letterhead auto-applied.

### User Story

> As a dentist, I want to generate prescriptions with my PRC license number and e-signature auto-applied so that I don't manually enter credentials per prescription.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Save Prescription | Primary | Saves prescription to patient record | `default` |
| Print | Secondary | Prints prescription with clinic letterhead | `outline` |
| Cancel | Ghost | Closes modal, discards unsaved changes | `ghost` |
| ✕ | Ghost | Closes modal | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container (centered) | — |
| `DialogHeader` | "Prescription" title + ✕ close | — |
| `Textarea` | Prescription body (medication, dosage, instructions) | `placeholder='Enter prescription...'` |
| `Input` | Medication name field | `type='text'` |
| `Input` | Dosage field | `type='text'` |
| `Input` | Frequency field | `type='text'` |
| `Input` | Duration field | `type='text'` |
| `Button` | "Save Prescription" | `variant='default'` |
| `Button` | "Print" | `variant='outline'` |
| `Button` | "Cancel" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Medication | text | Yes | Non-empty | Empty | `Prescription.medication` |
| Dosage | text | Yes | Non-empty | Empty | `Prescription.dosage` |
| Frequency | text | Yes | Non-empty | Empty | `Prescription.frequency` |
| Duration | text | No | — | Empty | `Prescription.duration` |
| Notes | textarea | No | Max 1000 chars | Empty | `Prescription.notes` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between`. "Prescription" title + ✕ close.
2. **PRC Section** — `px-6 py-3 bg-neutral-50 rounded-lg`. Read-only: "Dr. Maria Santos" name + "PRC-XXXXXXX" number + e-signature placeholder. Gray background to indicate non-editable.
3. **Form Fields** — `px-6 py-4 space-y-3`. Medication (Input), Dosage (Input), Frequency (Input), Duration (Input, optional label), Notes (Textarea).
4. **Footer** — `px-6 py-4 border-t flex justify-end gap-3`. "Cancel" ghost + "Print" outline + "Save Prescription" amber primary.

### Key Interactions

- **PRC auto-fill** — dentist's PRC license number and e-signature auto-applied from profile. Not editable in this modal.
- **Clinic letterhead** — clinic name, address, and contact info rendered on printed/exported prescriptions from clinic settings.
- **Print** → generates printable prescription with letterhead + PRC number + signature.
- **Multiple prescriptions** → save one, modal refreshes for another. All prescriptions stored against the current session.
- **Practice tier required** — FR8 is P1 (Practice tier and above). **Solo tier behavior:** modal opens but the form body is replaced by an inline upgrade prompt: illustration + "Prescriptions are available on the Practice tier." + "Learn More" ghost button (opens Settings/billing in a new tab, workspace state preserved) + "Maybe Later" ghost button (closes modal, returns to workspace). No form fields shown.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — form loads instantly. |
| **Empty** | Fresh form with all fields empty. PRC number and signature pre-filled (read-only). |
| **Error** | Toast: "Failed to save prescription" + Retry. |

### CRUD Interactions

#### Creating (prescription)
- **Trigger:** "Save Prescription" button
- **Opens:** N/A — saves from within the modal form
- **Form fields:** Medication, Dosage, Frequency, Duration, Notes (see Field Specs)
- **Validation:** Medication, Dosage, Frequency required. Non-empty.
- **On success:** Toast "Prescription saved." Modal form resets for another prescription. Prescription stored against the current patient session.
- **On cancel:** "Cancel" closes modal. If fields have been edited, no discard guard (prescription form is low-risk — user can re-enter quickly).

### What This Modal Does

- Generates prescriptions with auto-applied PRC credentials and e-signature
- Renders clinic letterhead on printed/exported documents
- Stores prescriptions against the current patient session

### What This Modal Does NOT Do

- Does NOT allow editing PRC credentials — those are set in the dentist profile (Settings)
- Does NOT manage medication databases or drug interaction checks — free-text entry only in Phase 1

### Wireframe

See [wireframes/modal-prescriptions.xml](wireframes/modal-prescriptions.xml)
