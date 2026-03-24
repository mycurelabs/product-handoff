---
screen: Consent Forms
type: modal
route: "N/A (modal within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Dialog
  - DialogHeader
  - Select
  - Button
wireframe: wireframes/modal-consent-forms.xml
wireframe_pages: [Default]
  - Default
---

## Modal: Consent Forms (of Dental Workspace)

**Route:** N/A (modal within `/workspace/:patientId`)
**POV:** A dentist capturing a patient's electronic signature on consent forms, waivers, or treatment agreements during a session.

### User Story

> As a dentist, I want to capture patient signatures on consent forms directly on the iPad screen so that signed documents are stored digitally with the patient record.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Save | Primary | Saves the signed consent form to patient record | `default` |
| Clear | Secondary | Clears the signature pad for re-signing | `outline` |
| Cancel | Ghost | Closes modal, discards unsigned form | `ghost` |
| ✕ | Ghost | Closes modal | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container (centered) | — |
| `DialogHeader` | "Consent Form" title + ✕ close | — |
| `Select` | Consent form type selector (Treatment Consent, Waiver, etc.) | — |
| **Signature Pad** | Draw-to-sign area for patient signature (custom canvas component) | — |
| **Signature Pad** | Draw-to-sign area for doctor signature (may auto-fill from profile) | — |
| `Button` | "Save" | `variant='default'` |
| `Button` | "Clear" | `variant='outline'` |
| `Button` | "Cancel" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Form Type | select | Yes | Must select from available form types | None | `ConsentForm.type` |
| Patient Signature | signature (canvas) | Yes | Non-empty (at least one stroke) | Empty | `ConsentForm.patient_signature` |
| Doctor Signature | signature (canvas) | No | Auto-filled from profile if available | Profile e-signature | `ConsentForm.doctor_signature` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between`. "Consent Form" title + ✕ close.
2. **Form Type** — `px-6 py-3`. Select dropdown ("Treatment Consent ▾").
3. **Consent Text** — `px-6 py-4`. Gray placeholder area (`h-[120px] bg-neutral-50 rounded-lg`): "Consent form content loads here."
4. **Patient Signature** — `px-6 py-3`. Label "Patient Signature" + bordered touch-to-sign area (`h-[100px] border-dashed border-2 rounded-lg`).
5. **Doctor Signature** — `px-6 py-3`. Label "Doctor Signature" + bordered area. May show "Auto-filled from profile" in muted text.
6. **Footer** — `px-6 py-4 border-t flex justify-end gap-3`. "Cancel" ghost + "Clear" outline + "Save" amber primary.

### Key Interactions

- **Signature pad** — touch-to-draw area. Patient signs directly on screen. Canvas captures strokes as image data.
- **"Clear"** → resets the currently active signature pad only (patient or doctor, whichever was last drawn on). Does NOT clear the other pad.
- **Doctor signature** → may auto-fill from the dentist's profile e-signature. Editable if manual signing is preferred.
- **"Save"** → stores the signed form with timestamp, form type, and both signatures. Attached to the current patient session.
- **Multiple consent forms** → save one, modal refreshes for another type.
- **Cancel with partial signature** → if any signature strokes have been captured (patient or doctor), tapping Cancel or ✕ triggers an AlertDialog: "Discard unsigned consent form?" with Cancel (stay) + Discard (close modal) buttons. Consistent with workspace wizard discard guard pattern. If no strokes captured, Cancel closes immediately.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — form loads instantly. |
| **Empty** | Form type selector + empty signature pads. |
| **Error** | Toast: "Failed to save consent form" + Retry. |

### CRUD Interactions

#### Creating (consent form)
- **Trigger:** "Save" button
- **Opens:** N/A — saves from within the modal
- **Form fields:** Form Type (select), Patient Signature (canvas), Doctor Signature (canvas) (see Field Specs)
- **Validation:** Form Type required. Patient Signature required (at least one stroke). Doctor Signature optional (auto-filled from profile).
- **On success:** Toast "Consent form saved." Modal form resets for another form type. Consent form stored with timestamp against patient session.
- **On cancel:** If any signature strokes captured → AlertDialog: "Discard unsigned consent form?" (Cancel/Discard). If no strokes → modal closes immediately.

### What This Modal Does

- Captures electronic signatures from both patient and dentist
- Stores signed consent forms with timestamps attached to patient records
- Supports multiple consent form types (treatment consent, waivers, etc.)

### What This Modal Does NOT Do

- Does NOT generate consent form templates — form content is pre-configured in Settings
- Does NOT handle insurance consent or third-party authorization forms — Phase 2

### Wireframe

See [wireframes/modal-consent-forms.xml](wireframes/modal-consent-forms.xml)
