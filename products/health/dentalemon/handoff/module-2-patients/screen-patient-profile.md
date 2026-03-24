---
screen: Patient Profile
type: screen
route: /patients/:id
module: "Module 2: Patient Management"
tier: 1
components:
  - Card
  - Avatar
  - Badge
  - Tabs
  - TabsList
  - TabsTrigger
  - TabsContent
  - Button
  - Alert
  - Separator
wireframe: wireframes/patient-profile.xml
wireframe_pages: [Default, Empty (New Patient)]
  - Default
  - "Empty (New Patient)"
---

## Screen: Patient Profile

**Route:** `/patients/:id`
**POV:** A dentist reviewing a patient's complete profile — demographics, dental history overview, and debt/balance status — before opening a workspace session or recording a payment.

### User Story

> As a dentist, I want to see a patient's demographics, dental history summary, and outstanding balance at a glance so that I'm prepared before starting their session.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Open Workspace | Primary | Opens Dental Workspace for this patient | `default` |
| Record Payment | Secondary | Opens inline payment recording form | `outline` |
| Edit Profile | Ghost | Opens edit form (demographics fields become editable) | `ghost` |
| Export Data | Ghost | Exports patient's complete data as CSV/JSON | `ghost` |
| Back | Ghost | Returns to Patient List | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Card` | Demographics card (name, contact, birthdate, gender, medical history) | — |
| `Card` | Dental History Overview card (total visits, last visit, treatments count, active conditions) | — |
| `Card` | Debt Summary card (total owed, last payment, aging breakdown) | — |
| `Avatar` | Patient photo (large) | `size='lg'` |
| `Badge` | Patient status (Active/Pending/Inactive) | `variant='outline'` |
| `Badge` | Debt aging badges (Current, 30 days, 60 days, 90+ days) | `variant='destructive'` for 90+ |
| `Tabs` | Profile tabs: Overview, Payment History | `defaultValue='overview'` |
| `TabsList` | Tab navigation | — |
| `TabsTrigger` | "Overview" tab trigger | `value='overview'` |
| `TabsTrigger` | "Payment History" tab trigger | `value='payment-history'` |
| `TabsContent` | Tab content panels | — |
| `Button` | "Open Workspace" CTA | `variant='default'` |
| `Button` | "Record Payment" | `variant='outline'` |
| `Button` | "Edit Profile" | `variant='ghost'` |
| `Button` | "Export Data" | `variant='ghost'` |
| `Button` | Back arrow | `variant='ghost'`, `size='icon'` |
| `Alert` | Overdue balance alert (if balance exceeds threshold or aging > 90 days) | `variant='warning'` |
| `Separator` | Between profile sections | — |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| First Name | text | Yes | Non-empty, max 100 chars | — | `Patient.first_name` |
| Last Name | text | Yes | Non-empty, max 100 chars | — | `Patient.last_name` |
| Contact Number | text | No | Philippine mobile format (09XX or +639XX) | — | `Patient.contact` |
| Email | text | No | Valid email format | — | `Patient.email` |
| Birthdate | date | Yes | Valid date, not in future | — | `Patient.birthdate` |
| Gender | select | Yes | Male / Female / Other | — | `Patient.gender` |
| Medical History Notes | textarea | No | Max 2000 chars | Empty | `Patient.medical_notes` |
| PWD | checkbox | No | Boolean | Unchecked | `Patient.is_pwd` |
| Senior Citizen | checkbox | No | Boolean | Unchecked | `Patient.is_senior` |

### Layout

1. **Header** — `px-6 py-4 flex items-center gap-4`. Back arrow button. Patient Avatar (large). Name (H2) + Status Badge. Right: "Open Workspace" primary + "Edit Profile" ghost + "Export Data" ghost.
2. **Alert Zone** — `px-6`. Overdue balance Alert (if applicable): "This patient has an outstanding balance of P X,XXX (90+ days overdue)."
3. **Tab Navigation** — `px-6 border-b`. Tabs: Overview | Payment History.
4. **Overview Tab Content** — `px-6 py-4 grid grid-cols-2 gap-6`.
   - Left column: Demographics Card (name, contact, birthdate, gender, email, medical history, PWD/Senior flags).
   - Right column: Dental History Overview Card (total visits, last visit date, total treatments, active conditions count) + Debt Summary Card (total owed, last payment date, aging breakdown: Current P X / 30d P X / 60d P X / 90+ P X with color-coded badges). "Record Payment" button below debt summary.
5. **Payment History Tab Content** — See Tab comment below.

### Key Interactions

- **"Open Workspace"** → navigates to `/workspace/:patientId`. Full-screen takeover (same as tapping a card on the Patient List).
- **"Edit Profile"** → demographics fields become editable inline. Save/Cancel buttons appear at the bottom. On save: toast "Profile updated."
- **"Record Payment"** → expands an inline form below the Debt Summary card: method select (required, default: Cash) + amount input + "Pay Exact" link + "Record" button. Validation: amount > 0, <= total outstanding (overpayment not allowed). On success: toast "Payment of P X recorded." Debt Summary card updates immediately (aging buckets recalculate). Payment appears in Payment History tab with "Manual" badge. On failure: toast "Payment failed to record" + Retry, form stays expanded with values preserved.
- **"Export Data"** → downloads complete patient data as CSV/JSON. Toast: "Data exported."
- **Overdue alert** → appears automatically when patient balance exceeds configurable threshold or aging > 90 days. Dismissable but reappears on next profile load.
- **PWD/Senior flags** → when checked on profile, discount auto-applied in Dental Workspace Payment Modal (Module 1). **Cross-module note:** the workspace Payment Modal (Module 1) also has inline PWD/Senior toggles so dentists can set the flag without leaving the workspace. Changes in either location sync to the same patient record.
- **Tab switching** → Overview (default) / Payment History. URL does not change — tab state is local.

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Open Workspace | Dental Workspace | `/workspace/:patientId` | Module 1: Dental Workspace |
| Back | Patient List | `/patients` | This module |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Header with avatar skeleton. Tab content shows shimmer cards. |
| **Empty (new patient, no history)** | Demographics populated from registration. Dental History card: "No visit history." Debt Summary: "No balance." |
| **Error** | Toast: "Failed to load patient profile" + Retry. |
| **Overdue** | Alert banner at top when balance > threshold or > 90 days. |

### CRUD Interactions

#### Editing (profile)
- **Trigger:** "Edit Profile" button
- **Opens:** Inline — demographics fields become editable in place
- **Form fields:** First Name, Last Name, Contact, Email, Birthdate, Gender, Medical History Notes, PWD, Senior Citizen (see Field Specs)
- **Validation:** First Name, Last Name, Birthdate, Gender required. Contact in PH mobile format. Email valid format.
- **On success:** Toast "Profile updated." Fields return to read-only.
- **On cancel:** Fields revert to last saved values. No confirmation needed (low-risk — no clinical data).

#### Recording Payment (manual)
- **Trigger:** "Record Payment" button on Debt Summary card
- **Opens:** Inline form below Debt Summary: Method (Cash/Bank Transfer/GCash/Maya) + Amount + "Record" button
- **Validation:** Amount > 0, <= total outstanding. Method required.
- **On success:** Toast "Payment recorded." Debt Summary updates immediately. Payment appears in Payment History tab. Record flagged as "manual."
- **On cancel:** Form collapses.

### What This Screen Does

- Displays patient demographics, dental history summary, and debt/balance status
- Shows aging breakdown for outstanding balances (current/30/60/90+ days)
- Provides overdue balance alerts
- Allows inline profile editing
- Supports manual payment recording outside of the workspace
- Provides one-tap entry to the Dental Workspace
- Offers full data export (CSV/JSON) per RA 10173

### What This Screen Does NOT Do

- Does NOT provide clinical charting or treatment management — that is Module 1: Dental Workspace
- Does NOT manage appointments — that is Module 3: Scheduling
- Does NOT show practice-wide reporting — that is Module 4: Reporting & Analytics
- Does NOT delete patients — archive only (from Patient List overflow menu)
