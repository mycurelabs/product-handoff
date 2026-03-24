---
screen: New Appointment
type: modal
route: null
module: "Module 3: Scheduling"
tier: 2
components:
  - Dialog
  - DialogHeader
  - Input
  - Card
  - DatePicker
  - Select
  - Textarea
  - Alert
  - Button
wireframe: inline-ascii
---

## Modal: New Appointment (of Calendar)

**Route:** N/A (modal within `/calendar`)
**POV:** A dentist or staff member scheduling a patient's next visit — selecting patient, date, time, and optionally a procedure type.

### User Story

> As a staff member, I want to create an appointment by searching for a patient and selecting a time slot so that the schedule is updated immediately with double-booking warnings.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Create Appointment | Primary | Creates the appointment and closes modal | `default` |
| Cancel | Ghost | Closes modal, discards form | `ghost` |
| ✕ | Ghost | Closes modal | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container | `className='max-w-[500px]'` |
| `DialogHeader` | "New Appointment" title + ✕ close | — |
| `Input` | Patient search field (autocomplete) | `type='search'`, `placeholder='Search patient...'` |
| `Card` | Patient search result cards (name, photo, contact last 4) | — |
| `DatePicker` | Appointment date | `defaultValue={prefilledDate}` |
| `Select` | Start time selector (time slots based on working hours + slot duration) | `defaultValue={prefilledTime}` |
| `Select` | Duration selector (30 min, 1 hr, 1.5 hr, 2 hr) | `defaultValue='30min'` |
| `Select` | Procedure type (optional — from treatment hierarchy) | `placeholder='Select procedure (optional)'` |
| `Textarea` | Notes (optional) | `placeholder='Appointment notes...'` |
| `Alert` | Double-booking warning | `variant='warning'` |
| `Button` | "Create Appointment" | `variant='default'` |
| `Button` | "Cancel" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Patient | search (autocomplete) | Yes | Must select existing patient from search results | Pre-filled if opened from patient context | `Appointment.patient_id` |
| Date | date | Yes | Valid date. Cannot be in the past (except today). | Today or pre-filled from tapped slot | `Appointment.date` |
| Start Time | select (time slots) | Yes | Must be within configured working hours | Pre-filled from tapped slot or first available | `Appointment.start_time` |
| Duration | select | Yes | 30 min / 1 hr / 1.5 hr / 2 hr | 30 min | `Appointment.duration` |
| Procedure Type | select | No | From treatment hierarchy (specialty → procedure) | None | `Appointment.procedure_type` |
| Notes | textarea | No | Max 500 chars | Empty | `Appointment.notes` |

### Layout

1. **Header** — "New Appointment" + ✕ close.
2. **Patient Search** — `space-y-2`. Search Input with autocomplete. Below: search result cards showing patient name, photo, contact last 4 digits. Tap to select. Selected patient shown as a chip with ✕ to deselect.
3. **Date & Time** — `flex gap-4`. DatePicker (half width) + Start Time Select (quarter width) + Duration Select (quarter width).
4. **Procedure** — `mt-4`. Procedure Type Select (optional, full width).
5. **Notes** — `mt-4`. Textarea (optional, full width).
6. **Double-Booking Warning** — Alert (warning) appears between form and footer if selected time overlaps: "This time overlaps with [Patient Name]'s appointment at [time]. Create anyway?"
7. **Footer** — `flex justify-end gap-3 pt-4 border-t`. "Cancel" ghost + "Create Appointment" primary.

### Key Interactions

- **Patient search** → debounced 300ms, same search engine as Patient List. Results show name, photo, contact last 4. Tap to select. If no patient found: "Patient not found — register first" link. **Registration return flow:** modal state (date, time, duration, procedure, notes) is preserved in local state. After patient registration completes in Module 2, the system auto-returns to this modal with the newly created patient pre-selected and all prior form state restored.
- **Pre-filled context** → when opened from a tapped time slot: date + time pre-filled. When opened from reschedule: patient + procedure pre-filled, date/time editable.
- **Double-booking detection** → on time selection, check for existing appointments at the same time. If overlap found: Warning Alert with existing patient name + time. Does NOT block creation — just warns. Dentist can override.
- **"Create Appointment"** → validates required fields → creates appointment → closes modal → calendar updates with new appointment block. Toast: "Appointment created for [Patient Name] at [time]."
- **Duration affects end time** → selecting duration calculates end_time from start_time. If end_time exceeds working hours: warning "This appointment extends past working hours ([end time])."
- **Working hours** → time slot options are constrained to configured working hours from Settings (Module 7). Default: 8:00 AM – 6:00 PM.
- **Walk-in context** → when opened from the walk-in flow (patient just registered), patient is pre-selected. Time defaults to "now" or next available slot. Date locked to today.

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| "Patient not found" link | Patient List (registration) | `/patients` | Module 2: Patient Management |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — form loads instantly. |
| **Empty** | Fresh form. Patient search empty. Date/time may be pre-filled from context. |
| **Double-booking detected** | Warning Alert between form and footer. Does not block creation. |
| **Error** | Toast: "Failed to create appointment" + Retry. Modal stays open with form data preserved. |
| **Success** | Modal closes. Toast: "Appointment created for [Patient Name] at [time]." Calendar updates with new appointment block. |

### CRUD Interactions

#### Creating (appointment)
- **Trigger:** "Create Appointment" button
- **Opens:** N/A — saves from within the modal form
- **Form fields:** See Field Specs
- **Validation:** Patient required (selected from search). Date required (not past). Start Time required (within working hours). Duration required.
- **On success:** Toast "Appointment created." Modal closes. Calendar updates with new block.
- **On cancel:** Modal closes. No appointment created.

#### Editing (appointment — opened from Calendar Edit action)
- **Trigger:** Calendar popover → Edit → modal opens pre-filled
- **Opens:** Same modal, all fields pre-filled and editable. Patient search shows the existing patient as a pre-selected chip.
- **Differences from Create:** Submit button label changes to "Save Changes." No new appointment created — updates the existing appointment in place. Patient field is editable (can reassign to a different patient).
- **Validation:** Same as Create.
- **On success:** Toast "Appointment updated." Modal closes. Calendar block reflects changes.
- **On cancel:** Modal closes. No changes to the appointment.

### What This Modal Does

- Creates appointments with patient, date, time, duration, and optional procedure type
- Provides patient search with autocomplete (same engine as Patient List)
- Detects and warns about double-booking conflicts
- Pre-fills context from tapped time slots, reschedule actions, or walk-in flow
- Validates against configured working hours

### What This Modal Does NOT Do

- Does NOT register new patients — routes to Module 2 if patient not found
- Does NOT send appointment confirmations — Phase 2 (FR19)
- Does NOT support recurring appointments — not in Phase 1
- Does NOT allow booking for other dentists — Phase 2 (FR16, multi-dentist)

### Wireframe

```
┌─────────────────────────────────────────┐
│ New Appointment                      ✕  │
├─────────────────────────────────────────┤
│ Patient*  [Search patient...       ]    │
│                                         │
│ ┌───────────────┐                       │
│ │ 📷 Joey M.    │ ← selected           │
│ │ Paciente ✕    │                       │
│ └───────────────┘                       │
│                                         │
│ Date*  [03/25/26]  Time* [10:00▾]      │
│                    Duration [30min▾]    │
│                                         │
│ Procedure  [Select procedure ▾]         │
│                                         │
│ Notes  [________________________]       │
│                                         │
│ ⚠ Overlaps with Patricia V. at 10:00  │
│                                         │
│ ┌────────┐  ┌────────────────────────┐ │
│ │ Cancel │  │  Create Appointment   │  │
│ └────────┘  └────────────────────────┘ │
└─────────────────────────────────────────┘
```