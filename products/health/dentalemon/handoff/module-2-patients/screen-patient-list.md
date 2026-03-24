---
screen: Patient List
type: screen
route: /patients
module: "Module 2: Patient Management"
tier: 1
components:
  - Input
  - Button
  - Patient Folder Card
  - Avatar
  - Badge
  - DropdownMenu
  - DropdownMenuItem
  - Dialog
  - Select
wireframe: wireframes/patient-list.xml
wireframe_pages: [Default, Empty, Search No Results]
  - Default
  - Empty
  - Search No Results
---

## Screen: Patient List

**Route:** `/patients`
**POV:** A dentist or staff member browsing their patient registry to find a patient, start a session, or register a new patient.

### User Story

> As a dentist, I want to browse and search my patient list using folder-style cards so that I can quickly find a patient and open their workspace or profile.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| + New Patient | Primary | Opens New Patient Registration modal | `default` |
| Patient Card (tap) | — | Opens Dental Workspace for that patient (full-screen takeover) | — |
| ⋮ (overflow per card) | Ghost | Opens dropdown: View Profile, Edit, Archive | `ghost` |
| Search | — | Filters patient cards by name match | — |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Input` | Search field (top of page) | `type='search'`, `placeholder='Search patients...'` |
| `Button` | "+ New Patient" CTA | `variant='default'` |
| **Patient Folder Card** | Custom SVG folder cards in responsive grid — CUSTOM COMPONENT, see epic | `patient={Patient}`, `onTap={() => void}`, `onOverflow={(action: 'profile' \| 'edit' \| 'archive') => void}`, `showInSession={boolean}`, `showDebtBadge={boolean}`, `width={130}`, `height={96}` |
| `Avatar` | Patient photo inside folder card tab | `size='md'` |
| `Badge` | Patient status (Active, Pending, Inactive) | `variant='outline'` |
| `DropdownMenu` | Overflow menu per card (⋮): View Profile, Edit, Archive | — |
| `DropdownMenuItem` | View Profile | — |
| `DropdownMenuItem` | Edit | — |
| `DropdownMenuItem` | Archive | `className='text-destructive'` |
| `Dialog` | New Patient Registration modal container | — |
| `Select` | Sort/filter controls (e.g., sort by name, status, last visit) | — |

> **Patient Folder Card SVG:** 130×96px base dimensions. The folder shape SVG assets are provided separately. Card scales responsively within the grid while maintaining aspect ratio (130:96 ≈ 1.35:1).

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Search | text (search input) | No | Debounced 300ms, partial name matching | Empty | — |

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Patients" heading + patient count badge. Right: "+ New Patient" primary button.
2. **Search & Filters** — `px-6 py-2 flex gap-4`. Search Input (flex-1). Sort Select (e.g., "Sort by: Last Visit"). Filter by status (All / Active / Pending).
3. **Card Grid** — `px-6 py-4 grid grid-cols-3 gap-4` (3 columns on iPad landscape, 2 on portrait). Each cell: Patient Folder Card (custom SVG). Cards show: photo/avatar in tab, decorative document lines, last name, first name + initial, gender icon + age, status badge + date, overflow ⋮.
4. **Empty State** — When no patients: centered illustration + "No patients yet. Add your first patient." + "+ New Patient" button.

### Key Interactions

- **Tap patient card body** → Dental Workspace opens for that patient (`/workspace/:patientId`). Full-screen takeover — sidebar disappears.
- **Tap ⋮ overflow on card** → DropdownMenu: View Profile (opens Patient Profile), Edit (opens registration modal pre-filled), Archive (confirmation AlertDialog).
- **Search** → debounced 300ms. Filters card grid in real-time. Partial name matching (first name, last name). Highlighted match text on cards.
- **"+ New Patient"** → opens New Patient Registration modal.
- **Sort** → alphabetical (A-Z, Z-A), by last visit date (newest first), by status.
- **Filter by status** → All, Active, Pending. Filter pills or segmented control.
- **Infinite scroll or pagination** → for large patient lists (500+ records). Load cards in batches of 30.
- **Walk-in shortcut** → from the Calendar module (Module 3), a "Walk-in" action routes here with the registration modal pre-opened.
- **Back navigation state preservation** → when the dentist returns from the workspace (back/close), the Patient List restores the previous search query, active filter, and scroll position. The list is not re-rendered from scratch. This is critical for busy practices navigating in and out dozens of times per day.
- **Near-duplicate disambiguation** → each Patient Folder Card shows birthdate year and last 4 digits of contact number below the name to help distinguish patients with similar names. This prevents tapping the wrong patient's card and opening the wrong workspace.
- **Archive action** → overflow menu → Archive triggers an AlertDialog: "Archive [Patient Name]? Archived patients are hidden from the active list but can be restored." with Cancel + Archive buttons. Archived patients are accessible via a filter (see below).
- **Archived filter** → the status filter expands to: All (excludes archived) / Active / Pending / Archived. "All" does NOT include archived patients by default — staff must explicitly select "Archived" to find them.
- **Restore from archive** → on an archived patient's card (visible under "Archived" filter), the overflow menu shows "Restore" instead of "Archive." Restore returns the patient to Active status.
- **"In Session" badge** → when a workspace is currently open for a patient, their card shows an "In Session" badge (blue, distinct from Active/Pending). This signals to front-desk staff that the dentist is currently seeing this patient. The badge clears when the workspace is closed.
- **SVG card accessibility** → each Patient Folder Card is wrapped in a focusable `<button>` element with `aria-label="[First Name] [Last Name], [Status], [Age]"`. The SVG folder shape is `aria-hidden="true"`. Overflow ⋮ button is positioned as a separate element above the SVG layer with min 44×44px touch target.
- **Debt aging badge accessibility** → debt aging badges on cards (if shown) include text labels ("90+ days") in addition to color. Color is never the sole differentiator per WCAG 1.4.1.

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Patient Card (tap) | Dental Workspace | `/workspace/:patientId` | Module 1: Dental Workspace |
| View Profile (overflow) | Patient Profile | `/patients/:id` | This module |
| + New Patient → "Create + Start Session" | Dental Workspace | `/workspace/:patientId` | Module 1: Dental Workspace |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Header renders. Card grid shows 6 skeleton folder cards (placeholder shapes). |
| **Empty (no patients)** | Illustration + "No patients yet. Add your first patient." + "+ New Patient" button. Search hidden. |
| **Empty (search, no results)** | "No patients match '[query]'" + "Clear search" link. |
| **Error** | Toast: "Failed to load patients" + Retry. Last known card grid remains visible if available. |

### CRUD Interactions

#### Archiving
- **Trigger:** Overflow menu (⋮) → Archive
- **Confirmation:** AlertDialog: "Archive [Patient Name]? Archived patients are hidden from the active list but can be restored." with Cancel + Archive buttons.
- **Soft vs hard delete:** Soft archive — patient hidden from default list, accessible via "Archived" filter.
- **On success:** Toast "Patient archived." Card removed from grid (unless Archived filter is active). Patient count badge decrements.
- **Blocked when:** Patient has an active workspace session ("In Session" badge). Cannot archive a patient currently being seen.

#### Restoring (from Archived filter)
- **Trigger:** Overflow menu (⋮) → Restore (visible only on archived patient cards)
- **Confirmation:** None — immediate restore.
- **On success:** Toast "Patient restored." Status changes to Active. Card reappears under Active/All filters.

### What This Screen Does

- Displays all patients as custom folder-style cards in a responsive grid
- Provides real-time search with partial name matching
- Offers sort and filter controls (name, status, last visit)
- Opens the Dental Workspace for a selected patient
- Provides "+ New Patient" entry point for registration
- Shows patient status (Active/Pending) and key info at a glance

### What This Screen Does NOT Do

- Does NOT display clinical treatment data — that is Module 1: Dental Workspace
- Does NOT manage appointments — that is Module 3: Scheduling
- Does NOT show revenue or reporting — that is Module 4: Reporting & Analytics
- Does NOT handle data import/migration — that is Module 6: Onboarding (FR15)
