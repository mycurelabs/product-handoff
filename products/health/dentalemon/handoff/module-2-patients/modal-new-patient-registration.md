---
screen: New Patient Registration
type: modal
route: null
module: "Module 2: Patient Management"
tier: 1
components:
  - Dialog
  - DialogHeader
  - Input
  - DatePicker
  - Select
  - Textarea
  - Checkbox
  - Collapsible
  - Button
  - Alert
wireframe: inline-ascii
---

## Modal: New Patient Registration (of Patient List)

**Route:** N/A (modal within `/patients`)
**POV:** A dentist or staff member registering a new patient with minimal friction — quick capture of essential info with optional fields expandable for completeness.

### User Story

> As a dentist, I want to register a new patient in under 2 minutes with just a name, contact, and birthdate so that I can start their session immediately without front-loading the form.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Create Patient | Primary | Creates patient record and returns to Patient List | `default` |
| Create + Start Session | Secondary | Creates patient record and opens Dental Workspace immediately | `outline` |
| Cancel | Ghost | Closes modal, discards form | `ghost` |
| ✕ | Ghost | Closes modal | `ghost` |
| Show More Fields | Ghost | Expands optional fields section | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Dialog` | Modal container (centered) | `className='max-w-[500px]'` |
| `DialogHeader` | "New Patient" title + ✕ close | — |
| `Input` | First Name, Last Name, Contact Number fields | `type='text'` |
| `DatePicker` | Birthdate field | — |
| `Select` | Gender selector (Male / Female / Other) | — |
| `Input` | Email field (optional section) | `type='email'` |
| `Textarea` | Medical History Notes (optional section) | `placeholder='Any known allergies, conditions...'` |
| `Checkbox` | PWD flag | — |
| `Checkbox` | Senior Citizen flag | — |
| `Collapsible` | Optional fields section (expanded via "Show More Fields") | `defaultOpen={false}` |
| `Button` | "Create Patient" | `variant='default'` |
| `Button` | "Create + Start Session" | `variant='outline'` |
| `Button` | "Cancel" | `variant='ghost'` |
| `Alert` | Duplicate detection warning | `variant='warning'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| First Name | text | Yes | Non-empty, max 100 chars | Empty | `Patient.first_name` |
| Last Name | text | Yes | Non-empty, max 100 chars | Empty | `Patient.last_name` |
| Contact Number | text | Yes | Philippine mobile format (09XX or +639XX), 11 digits | Empty | `Patient.contact` |
| Birthdate | date | Yes | Valid date, not in future | Empty | `Patient.birthdate` |
| Gender | select | Yes | Male / Female / Other | None | `Patient.gender` |
| Email | text | No | Valid email format | Empty | `Patient.email` |
| Medical History Notes | textarea | No | Max 2000 chars | Empty | `Patient.medical_notes` |
| PWD | checkbox | No | Boolean | Unchecked | `Patient.is_pwd` |
| Senior Citizen | checkbox | No | Boolean | Unchecked | `Patient.is_senior` |

### Layout

1. **Header** — "New Patient" + ✕ close.
2. **Required Fields** — `space-y-4`. First Name Input, Last Name Input, Contact Number Input, Birthdate DatePicker, Gender Select. All visible by default.
3. **"Show More Fields"** — Ghost button/link below required fields. On tap: Collapsible expands to show optional fields.
4. **Optional Fields** — `space-y-4` (collapsed by default). Email Input, Medical History Notes Textarea, PWD Checkbox, Senior Citizen Checkbox.
5. **Duplicate Warning** — Alert (warning) appears between form and footer if fuzzy name match detected: "A patient named '[Name]' already exists. Continue anyway?"
6. **Footer** — `flex justify-end gap-3 pt-4 border-t`. "Cancel" ghost + "Create Patient" primary + "Create + Start Session" outline.

### Key Interactions

- **Progressive disclosure** → only 5 required fields shown initially. Optional fields hidden behind "Show More Fields." This keeps the form under 2 minutes for the common case.
- **Duplicate detection** → on Last Name blur, fuzzy match against existing patients. If match > 80%: warning Alert appears showing the matching patient's **photo/avatar, full name, contact last 4 digits, and last visit date** so staff can confirm whether it's the same person. Alert includes "View Existing" link (opens Patient Profile in new context) + "Continue Anyway" text. Does NOT block creation — just warns with enough context to decide.
- **"Create Patient"** → validates required fields → creates record → closes modal → new Patient Folder Card appears in the grid. Toast: "Patient created."
- **"Create + Start Session"** → validates → **button shows spinner + "Creating..."** while the local write completes (prevents navigating to workspace with uncommitted record) → on write success: closes modal → navigates to `/workspace/:patientId`. PWD/Senior flags from the form are immediately available in the workspace session.
- **"Cancel" / ✕** → if form has been edited, no discard guard (registration form is quick to re-enter). Closes immediately.
- **Walk-in context** → when opened from Calendar module's walk-in action, the modal pre-fills today's date context and shows a note: "Walk-in patient — will be added to today's schedule."

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Create Patient | Patient List (new card appears) | `/patients` | This module |
| Create + Start Session | Dental Workspace | `/workspace/:patientId` | Module 1: Dental Workspace |
| View Existing (duplicate warning) | Patient Profile | `/patients/:id` | This module |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — form loads instantly. |
| **Empty** | Fresh form, all required fields empty. Optional fields collapsed. |
| **Duplicate detected** | Warning Alert appears below form: "A patient named '[Name]' already exists." with "View Existing" link + "Continue Anyway" text. |
| **Error** | Toast: "Failed to create patient" + Retry. Modal stays open with form data preserved. |

### CRUD Interactions

#### Creating (patient)
- **Trigger:** "Create Patient" or "Create + Start Session" button
- **Opens:** N/A — saves from within the modal form
- **Form fields:** See Field Specs above
- **Validation:** First Name, Last Name, Contact, Birthdate, Gender required. Contact in PH mobile format (09XX, 11 digits). Birthdate not in future.
- **On success (Create):** Toast "Patient created." Modal closes. New card appears in Patient List grid.
- **On success (Create + Start Session):** Toast "Patient created." Modal closes. Dental Workspace opens for new patient.
- **On cancel:** Modal closes. No record created.
- **Blocked when:** N/A — no blocking conditions. Duplicate detection warns but does not block.

### What This Modal Does

- Registers a new patient with minimal required fields (name, contact, birthdate, gender)
- Supports progressive field expansion for optional data (email, medical history, PWD/Senior flags)
- Detects potential duplicate patients by fuzzy name matching
- Offers immediate workspace entry via "Create + Start Session"

### What This Modal Does NOT Do

- Does NOT import patient data from external systems — that is Module 6: Onboarding (FR15)
- Does NOT capture clinical data (treatment history, charting) — that is Module 1
- Does NOT schedule an appointment — that is Module 3: Scheduling

### Wireframe

```
┌─────────────────────────────────────────┐
│ New Patient                          ✕  │
├─────────────────────────────────────────┤
│ First Name*      [________________]     │
│ Last Name*       [________________]     │
│ Contact No.*     [09_________]          │
│ Birthdate*       [__/__/____]           │
│ Gender*          [Select ▾]             │
│                                         │
│ ▸ Show More Fields                      │
│                                         │
│ ⚠ A patient named "Herrera" already    │
│   exists. View Existing · Continue      │
│                                         │
│ ┌────────┐ ┌──────────────┐ ┌────────┐ │
│ │ Cancel │ │Create Patient│ │+Session│ │
│ └────────┘ └──────────────┘ └────────┘ │
└─────────────────────────────────────────┘
```
