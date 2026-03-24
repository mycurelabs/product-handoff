---
screen: Onboarding Wizard
type: screen
route: /onboarding
module: "Module 6: Onboarding"
tier: 3
components:
  - Card
  - Progress
  - Input
  - Textarea
  - Table
  - Button
wireframe: wireframes/onboarding-wizard.xml
wireframe_pages: [Welcome, Step 1 - Clinic Setup, Step 3 - Fee Schedule, Completion]
  - Welcome
  - Step 1 - Clinic Setup
  - Step 3 - Fee Schedule
  - Completion
---

## Screen: Onboarding Wizard

**Route:** `/onboarding`
**POV:** A dentist launching Dentalemon for the first time — setting up their clinic, profile, and fee schedule before seeing their first patient.

### User Story

> As a first-time dentist user, I want to set up my clinic in under 10 minutes with guided steps so that I can start seeing patients immediately.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Continue | Primary | Advances to next wizard step | `default` |
| Back | Ghost | Returns to previous step | `ghost` |
| Skip (on optional steps) | Ghost | Skips current step, advances to next | `ghost` |
| Starting Fresh | Primary | Selects the "new practice" onboarding path | `default` |
| Migrating | Secondary | Selects the migration path → routes to Data Import | `outline` |
| Complete Setup | Primary | Finishes wizard → redirects to Patient List | `default` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Card` | Wizard step container (centered, max-width) | `className='max-w-[600px] mx-auto'` |
| `Progress` | Step progress bar (e.g., Step 2 of 4) | `value={stepPercent}` |
| `Input` | Clinic name, address, contact, PRC number, dentist name fields | `type='text'` |
| `Textarea` | Clinic address (multi-line) | — |
| `Input` | PRC license number | `type='text'`, `placeholder='PRC-XXXXXXX'` |
| `Table` | Treatment fee schedule (Treatment, Default Price) | — |
| `Input` | Fee amount per treatment (inline editable) | `type='number'` |
| `Button` | "Continue" | `variant='default'` |
| `Button` | "Back" | `variant='ghost'` |
| `Button` | "Skip" | `variant='ghost'` |
| `Button` | "Starting Fresh" / "Migrating" (routing step) | `variant='default'` / `variant='outline'` |
| `Button` | "Complete Setup" | `variant='default'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Clinic Name | text | Yes | Non-empty, max 200 chars | Empty | `Clinic.name` |
| Clinic Address | textarea | Yes | Non-empty | Empty | `Clinic.address` |
| Clinic Contact | text | Yes | PH mobile format | Empty | `Clinic.contact` |
| Dentist Name | text | Yes | Non-empty | Empty | `Dentist.name` |
| PRC License Number | text | Yes | PRC format validation (PRC-XXXXXXX) | Empty | `Dentist.prc_number` |
| E-Signature | signature (canvas) | No | Optional — can be set later in Settings | Empty | `Dentist.signature` |
| Security Question | select | Yes | Must select from predefined list (e.g., "Mother's maiden name", "First pet's name", "Childhood street") | None | `Dentist.security_question` |
| Security Answer | text | Yes | Non-empty, max 200 chars, case-insensitive comparison on verification | Empty | `Dentist.security_answer` |
| Treatment Fee (per row) | number | Yes | ≥ 0 | Pre-populated market average | `FeeSchedule.price` |

### Layout

1. **Welcome** — Dr. Lemon mascot illustration + "Welcome to Dentalemon" + tagline. Two path buttons: "Starting Fresh" (primary) + "Migrating from another system" (outline).
2. **Step 1: Clinic Setup** — Card with: Clinic Name, Address, Contact fields. Progress bar: Step 1 of 4.
3. **Step 2: Dentist Profile** — Card with: Dentist Name, PRC License Number, optional E-Signature capture pad. Progress bar: Step 2 of 4.
4. **Step 3: Treatment Fee Schedule** — Card with: table of common treatments + default prices (pre-populated). Each price is inline-editable. "These are default prices. You can change them anytime in Settings." Progress bar: Step 3 of 4.
5. **Step 4: First Patient** — Card with: minimal patient registration form (same as Module 2 New Patient Registration, but embedded inline). "Add your first patient to get started." Progress bar: Step 4 of 4.
6. **Completion** — Dr. Lemon celebration illustration + "You're all set!" + "Complete Setup" button → redirects to Patient List.

### Key Interactions

- **Routing step** → "Starting Fresh" proceeds through Steps 1-4 normally. "Migrating" routes to Data Import (`/onboarding/import`) first. After import completes, wizard resumes at **Step 3 (Fee Schedule)**. Step 4 (First Patient) is auto-skipped if patients were imported — the imported patients are already in the database.
- **Progress persistence** → wizard state saved to local storage after each step. If user quits mid-wizard, they resume at the last completed step on next launch.
- **Fee schedule pre-population** → common Philippine dental treatments (Filling, Cleaning, Extraction, Crown, Root Canal, etc.) with market-average prices. All editable. New treatments can be added later in Settings.
- **PRC validation** → PRC license number validated on format (PRC-XXXXXXX pattern). Not a live API check.
- **Step 4 (First Patient)** → uses same field set as Module 2 New Patient Registration. On submit, patient is created and immediately available in Patient List.
- **"Complete Setup"** → saves all wizard data, creates the clinic + dentist + fee schedule + first patient records, and redirects to `/patients` (Patient List).
- **Skip** → available on Step 4 (first patient is optional — dentist can add patients later). Not available on Steps 1-3 (clinic + profile + fees are required for core functionality).

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Migrating | Data Import | `/onboarding/import` | This module |
| Complete Setup | Patient List | `/patients` | Module 2: Patient Management |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — wizard loads instantly from local state. |
| **Resuming** | If wizard was interrupted, shows the last completed step with a "Welcome back" message and "Continue where you left off" button. |
| **Error (save failure)** | Toast: "Failed to save. Please try again." Wizard stays on current step. Data preserved. |
| **Already completed** | If onboarding was already completed, navigating to `/onboarding` redirects to `/patients`. |

### CRUD Interactions

#### Creating (clinic + dentist + fee schedule + first patient)
- **Trigger:** "Complete Setup" button on final step
- **Opens:** N/A — commits all wizard data at once
- **Form fields:** See Field Specs (7 fields across 4 steps)
- **Validation:** Steps 1-3 required. Step 4 optional (skippable).
- **On success:** All records created (Clinic, Dentist profile, FeeSchedule, optionally first Patient). Toast: "Setup complete!" Redirects to `/patients`.
- **On cancel:** Wizard can be abandoned at any time — progress persisted. Resumable on next launch.
- **On error:** Toast: "Failed to save. Please try again." Stays on current step. Data preserved.

### What This Screen Does

- Guides first-time setup: clinic info, dentist profile with PRC, treatment fee schedule, first patient
- Routes between "starting fresh" and "migrating" onboarding paths
- Pre-populates fee schedule with common dental treatments
- Persists wizard progress for interrupted sessions
- Features Dr. Lemon mascot on welcome and completion screens only

### What This Screen Does NOT Do

- Does NOT handle data import — that is the Data Import screen
- Does NOT provide in-app tips during regular use — Dr. Lemon is onboarding only in Phase 1
- Does NOT manage ongoing clinic settings — that is Module 7: Settings
- Does NOT include tutorial video library — FR13.3 is deferred (content creation is a marketing task)
- Does NOT provide Dr. Lemon guided walkthrough tips throughout the app — Phase 1 delivers mascot on onboarding screens only; contextual tips are Phase 2
