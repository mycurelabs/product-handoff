---
screen: Settings
type: screen
route: /settings
module: "Module 7: Settings"
tier: 3
components:
  - Tabs
  - TabsTrigger
  - Input
  - Textarea
  - Table
  - Button
  - SignatureCanvas
  - InputOTP
  - Progress
  - Badge
  - Select
  - Switch
  - Alert
  - AlertDialog
wireframe: wireframes/settings.xml
wireframe_pages: [Default (Fees Tab), Staff Role (Access Denied), Storage Full (Backup Tab)]
---

## Screen: Settings

**Route:** `/settings`
**POV:** A dentist-owner configuring their clinic's operational settings — backup, fees, working hours, profile, and notifications.

### User Story

> As a dentist-owner, I want to manage all clinic settings from one screen so that I can configure backup, pricing, working hours, and my professional profile in one place.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Save (per section) | Primary | Saves changes for the current settings section | `default` |
| Export All Data | Secondary | Downloads complete clinic data as CSV/JSON | `outline` |
| Back Up Now | Secondary | Triggers an immediate cloud backup | `outline` |
| Buy More Storage | Ghost | Opens external link to storage expansion purchase | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Tabs` | Settings sections: Clinic, Profile, Fees, Backup, Notifications | `defaultValue='clinic'` |
| `TabsTrigger` | Clinic tab | `value='clinic'` |
| `TabsTrigger` | Profile tab | `value='profile'` |
| `TabsTrigger` | Fees tab | `value='fees'` |
| `TabsTrigger` | Backup tab | `value='backup'` |
| `TabsTrigger` | Notifications tab | `value='notifications'` |
| `Input` | Clinic name, address, contact, dentist name, PRC number | `type='text'` |
| `Textarea` | Clinic address (multi-line) | — |
| `Input` | PRC license number | `type='text'` |
| `Table` | Treatment fee schedule (Treatment, Specialty, Default Price) | `aria-label='Treatment Fee Schedule'` |
| `Input` | Fee amount (inline editable per row) | `type='number'` |
| `Button` | "+ Add Treatment" (add custom treatment to fee schedule) | `variant='outline'` |
| `Button` | Remove treatment (trash icon, custom treatments only) | `variant='ghost'`, `size='icon'`, `className='text-destructive'` |
| **SignatureCanvas** | E-Signature capture pad on Profile tab (custom canvas component — same as Module 1 Consent Forms) | — |
| `InputOTP` | PIN change fields on Profile tab (current + new + confirm) | `maxLength={6}` |
| `Progress` | Cloud storage usage bar (used / total) | `value={storagePercent}` |
| `Badge` | Backup status: "Up to date" (green) / "Syncing" (amber) / "Offline" (muted) | `variant='outline'` |
| `Select` | Working hours start time | — |
| `Select` | Working hours end time | — |
| `Select` | Appointment slot duration (30 min / 1 hr) | `defaultValue='30min'` |
| `Switch` | Notification toggles (email receipts, appointment reminders, etc.) | — |
| `Button` | "Save" (per section footer) | `variant='default'` |
| `Button` | "Export All Data" | `variant='outline'` |
| `Button` | "Back Up Now" | `variant='outline'` |
| `Button` | "Buy More Storage" | `variant='ghost'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Clinic Name | text | Yes | Non-empty | From onboarding | `Clinic.name` |
| Clinic Address | textarea | Yes | Non-empty | From onboarding | `Clinic.address` |
| Clinic Contact | text | Yes | PH mobile format | From onboarding | `Clinic.contact` |
| Dentist Name | text | Yes | Non-empty | From onboarding | `Dentist.name` |
| PRC License Number | text | Yes | PRC format (PRC-XXXXXXX) | From onboarding | `Dentist.prc_number` |
| E-Signature | signature (canvas) | No | — | From onboarding | `Dentist.signature` |
| Treatment Fee (per row) | number | Yes | ≥ 0 | Market average | `FeeSchedule.price` |
| Working Hours Start | select (time) | Yes | Must be < end time | 8:00 AM | `Clinic.working_hours_start` |
| Working Hours End | select (time) | Yes | Must be > start time | 6:00 PM | `Clinic.working_hours_end` |
| Slot Duration | select | Yes | 30 min / 1 hr | 30 min | `Clinic.slot_duration` |
| Inactivity Timeout | select | Yes | 5 min / 10 min / 15 min / 30 min / Never | 5 min | `Clinic.inactivity_timeout` |
| Email Receipts | switch | No | Boolean | ON | `Notifications.email_receipts` |
| Appointment Reminders | switch | No | Boolean | ON | `Notifications.appointment_reminders` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between`. Left: "Settings" heading. Right: "Export All Data" outline button.
2. **Tabs** — `px-6 border-b`. Clinic | Profile | Fees | Backup | Notifications.
3. **Clinic Tab** — Clinic Name, Address, Contact fields. Working Hours (start/end time selects + slot duration). Inactivity Timeout select (5/10/15/30 min / Never — controls auto-lock per Module 8 Login). Save button.
4. **Profile Tab** — Dentist Name, PRC License Number, E-Signature capture pad. Save button.
5. **Fees Tab** — Treatment fee schedule table (Treatment, Specialty, Price — inline editable). "+ Add Treatment" button. Note: "Price changes apply to future treatments only." Save button.
6. **Backup Tab** — Cloud backup status card (last backup, sync status badge). Storage usage Progress bar (e.g., "4.2 GB of 20 GB used"). "Back Up Now" button. "Buy More Storage" link (external). Full restore instructions link.
7. **Notifications Tab** — Toggle switches for each notification type (email receipts, appointment reminders, storage warnings, license reminders). Save button.

### Key Interactions

- **Tab navigation** → each tab is an independent settings section. Changes within one tab don't affect others.
- **Save per section** → each tab has its own Save button. Unsaved changes trigger a warning badge on the tab name. Navigating away from a dirty tab: toast "You have unsaved changes in [tab name]."
- **Fee schedule editing** → inline price editing on each row. "+ Add Treatment" opens an inline form (treatment name + specialty + price). Fee changes apply prospectively — existing breakdown table rows keep original prices.
- **"Export All Data"** → downloads complete clinic data (patients, treatments, payments, appointments) as CSV/JSON archive. Per RA 10173 data portability. Confirmation dialog: "Export all clinic data? This may take a moment for large datasets."
- **"Back Up Now"** → triggers immediate cloud backup. Progress indicator during backup. Toast: "Backup complete" or "Backup failed — will retry automatically."
- **"Buy More Storage"** → opens external purchase page in a new tab. Does not navigate away from the app.
- **Storage warnings** → when storage usage > 80%: amber Progress bar + alert text "Storage nearly full." When > 95%: destructive Progress bar + "Buy More Storage" becomes prominently visible.
- **Dentist-owner only** → if staff navigates to `/settings`: "Access restricted to Dentist-Owner." Same pattern as Module 5.
- **PRC number changes** → validated on save. If format is invalid: inline error below the field.
- **Tab switching with unsaved changes** → switching to a different tab with unsaved changes triggers: "Save changes to [tab name]?" with Save / Discard / Cancel (stays on current tab). Prevents data loss.
- **Remove custom treatment** → custom treatments added via "+ Add Treatment" have a Remove action (trash icon) in the fee schedule table. Default treatments (pre-populated) cannot be removed. Confirmation: AlertDialog "Remove [Treatment Name] from fee schedule? This won't affect existing treatment records." Cancel + Remove (destructive).
- **Dentist-owner PIN change** → Profile tab includes a "Change PIN" link that reveals a PIN update form (current PIN + new PIN + confirm new PIN). This is the dentist-owner's path to change their own login PIN. Staff PINs are managed in Module 5.
- **Export format selection** → "Export All Data" opens a confirmation dialog with format selector (CSV or JSON) before downloading. Both formats include the same data.
- **Working hours limitation** → Phase 1 supports start/end time only (no lunch breaks, no day-of-week schedules). Note in Fees tab: "Break times and day-specific hours coming in a future update."

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Tabs render. Active tab content shows shimmer fields. |
| **Unsaved changes** | Tab name shows warning badge (amber dot). Toast on navigate-away: "Unsaved changes in [tab]." |
| **Staff role (access denied)** | Full screen: "Access restricted to Dentist-Owner." Back to previous screen. |
| **Backup offline** | Backup tab shows: "Offline — backup will sync when connected." Badge: "Offline" (muted). "Back Up Now" disabled. |
| **Storage full** | Backup tab: destructive Progress bar. Alert: "Storage full. Back up will fail until storage is expanded." |
| **Empty** | N/A — Settings always populated from onboarding data. Tabs always have content. |
| **Error** | Toast per section: "Failed to save [section name]" + Retry. |

### CRUD Interactions

#### Editing (clinic settings)
- **Trigger:** Edit any field + tap "Save"
- **Opens:** Inline — fields are always editable
- **Validation:** Per Field Specs (PRC format, time range, non-empty required fields)
- **On success:** Toast "Settings saved." Fields return to display state.
- **On cancel:** N/A — no explicit cancel. Navigate away triggers unsaved-changes warning.

#### Adding (custom treatment to fee schedule)
- **Trigger:** "+ Add Treatment" button on Fees tab
- **Opens:** Inline form row at bottom of fee schedule table
- **Form fields:** Treatment Name (text, required), Specialty (select, required), Price (number, required)
- **On success:** New row added to fee schedule. Toast "Treatment added."
- **On cancel:** Inline form row collapses.

### What This Screen Does

- Manages clinic info (name, address, contact, working hours)
- Manages dentist profile (name, PRC number, e-signature)
- Configures treatment fee schedule (edit prices, add custom treatments)
- Monitors cloud backup status and storage usage
- Provides full data export (CSV/JSON) for data portability
- Configures notification preferences
- Triggers manual backup

### What This Screen Does NOT Do

- Does NOT manage staff accounts — that is Module 5: Staff Management
- Does NOT handle license purchase or renewal — external
- Does NOT process storage expansion payments — links to external purchase
- Does NOT manage patient data — that is Module 2: Patient Management
- Does NOT provide analytics — that is Module 4: Reporting & Analytics
