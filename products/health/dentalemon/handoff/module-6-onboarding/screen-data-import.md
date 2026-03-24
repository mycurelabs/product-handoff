---
screen: Data Import
type: screen
route: /onboarding/import
module: "Module 6: Onboarding"
tier: 3
components:
  - Card
  - Button
  - Table
  - TableRow
  - Badge
  - Alert
  - Progress
wireframe: inline-ascii
---

## Screen: Data Import

**Route:** `/onboarding/import`
**POV:** A Software Switcher dentist migrating patient records from their previous system (MyMeds, MYCURE, etc.) via CSV upload.

### User Story

> As a dentist switching from another system, I want to import my existing patient records via CSV so that I don't have to re-enter hundreds of patients manually.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Upload CSV | Primary | Opens file picker to select CSV file | `default` |
| Import | Primary | Commits validated records to the database | `default` |
| Cancel | Ghost | Returns to Onboarding Wizard without importing | `ghost` |
| Download Template | Ghost | Downloads a blank CSV template with correct column headers | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Card` | Import container (centered, max-width) | `className='max-w-[700px] mx-auto'` |
| `Button` | "Upload CSV" (file picker trigger) | `variant='default'` |
| `Button` | "Download Template" | `variant='ghost'` |
| `Table` | Preview table showing parsed CSV rows with validation status | `aria-label='Import Preview'` |
| `TableRow` | Data rows with validation indicators | — |
| `Badge` | Row status: Valid (green) / Error (red) / Warning (amber) | `variant='outline'` |
| `Alert` | Summary: "X of Y records valid. Z errors found." | `variant='default'` or `variant='destructive'` |
| `Progress` | Import progress bar (during commit) | `value={importPercent}` |
| `Button` | "Import" (commit valid records) | `variant='default'` |
| `Button` | "Cancel" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| CSV File | file upload | Yes | Must be .csv format. Max 10MB. | None | — |

### Layout

1. **Header** — `px-6 py-4`. "Import Patient Records" heading + "Download Template" ghost link.
2. **Upload Zone** — `px-6 py-8 border-2 border-dashed rounded-lg text-center`. Drag-and-drop zone + "Upload CSV" button. Accepts .csv files only.
3. **Preview Table** (after upload) — `px-6 py-4`. Shows parsed rows: Name, Contact, Birthdate, Gender, Status (Valid/Error badge). Error rows highlighted with red border + specific error message (e.g., "Missing birthdate"). Summary Alert above table: "142 of 150 records valid. 8 errors found — review below."
4. **Footer** — `px-6 py-4 flex justify-between`. "Cancel" ghost (returns to wizard). "Import [N] valid records" primary (disabled if 0 valid).

### Key Interactions

- **Upload CSV** → file picker opens. After selection: CSV parsed, preview table rendered with validation status per row.
- **Validation** → required columns: Name (first + last), Contact, Birthdate. Optional: Gender, Email, Medical Notes. Missing required fields → Error badge on row. Invalid format (e.g., bad date) → Warning badge.
- **Error rows** → highlighted red. Clickable to expand error detail. Error rows are skipped during import — only valid rows are committed.
- **"Import [N] valid records"** → Progress bar shows import progress. Each record created in local database. On complete: toast "Imported [N] patient records." Redirects back to Onboarding Wizard to continue remaining steps.
- **"Download Template"** → downloads a .csv file with correct column headers and 2 example rows. Helps dentists prepare their data.
- **"Cancel"** → returns to Onboarding Wizard without importing. No data changes.
- **Large imports** → for 500+ records, show estimated time. Import runs in background with progress bar.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — screen loads instantly. |
| **Empty (no file uploaded)** | Upload zone with drag-and-drop illustration + "Upload CSV" button. |
| **Preview (file uploaded)** | Preview table with validation results. Import button enabled if ≥1 valid record. |
| **Importing** | Progress bar. Cancel disabled. "Importing [N] records..." |
| **Complete** | Toast: "Imported [N] records." Auto-redirect to Onboarding Wizard. |
| **Error (parse failure)** | Alert: "Could not read this file. Please check the format and try again." + "Download Template" link. |

### CRUD Interactions

#### Importing (bulk patient creation)
- **Trigger:** "Import" button
- **Opens:** N/A — inline progress bar
- **Validation:** Only valid rows are imported. Error rows skipped with count shown.
- **On success:** Toast "Imported [N] patient records." Redirect to Onboarding Wizard.
- **On cancel:** N/A — import cannot be cancelled mid-progress.
- **On error:** Import uses **batch-commit with rollback** — if any write fails, ALL records in the current batch are rolled back. Toast: "Import failed. No records were imported. Please try again." User can re-upload the same file. This ensures data integrity (no partial imports that leave the database in an inconsistent state).

### What This Screen Does

- Accepts CSV file upload for bulk patient record import
- Validates each row against required fields and format rules
- Shows preview with per-row validation status before committing
- Imports only valid records, skipping error rows
- Provides a downloadable CSV template with correct column headers

### What This Screen Does NOT Do

- Does NOT import treatment history or financial records — patient demographics only
- Does NOT connect to external APIs (MyMeds, MYCURE) — CSV manual export/import only
- Does NOT support ongoing sync — one-time import during onboarding
- Does NOT handle CRO-assisted migrations — those are operational, not product UI

### Wireframe

```
┌──────────────────────────────────────────────────┐
│ Import Patient Records       Download Template   │
├──────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────────────────────────────────────┐    │
│  │        📄 Drag CSV here or               │    │
│  │           [Upload CSV]                   │    │
│  └──────────────────────────────────────────┘    │
│                                                  │
│  ⚠ 142 of 150 records valid. 8 errors found.   │
│                                                  │
│  Name          Contact     Birthdate   Status    │
│  ─────────────────────────────────────────────   │
│  Joey Paciente 0917...1234 02/06/24    ✓ Valid  │
│  Patricia C.   0918...5678 02/03/24    ✓ Valid  │
│  (missing)     0919...0000 —           ✗ Error  │
│                                                  │
│  [Cancel]              [Import 142 valid records]│
└──────────────────────────────────────────────────┘
```
