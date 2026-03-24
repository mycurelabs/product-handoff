---
screen: Staff Management
type: screen
route: /staff
module: "Module 5: Staff Management"
tier: 3
components:
  - Table
  - TableHeader
  - TableRow
  - Badge
  - Button
  - Input
  - Select
  - Alert
  - AlertDialog
wireframe: inline-ascii
---

## Screen: Staff Management

**Route:** `/staff`
**POV:** A dentist-owner managing their clinic's staff accounts — adding new staff, assigning roles, and controlling access permissions.

### User Story

> As a dentist-owner, I want to manage staff accounts with role-based access so that my secretary can handle scheduling and patient lookup without accessing clinical notes or financial analytics.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| + Add Staff | Primary | Opens inline form to create a new staff account | `default` |
| Edit (per row) | Ghost | Opens inline edit for staff name, role, PIN | `ghost` |
| Deactivate (per row) | Destructive | Deactivates staff account (revokes access) | `destructive` |
| Reactivate (per row) | Ghost | Restores a deactivated staff account | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Table` | Staff list (Name, Role, Status, Last Login, Actions) | `aria-label='Staff Accounts'` |
| `TableHeader` | Column headers | — |
| `TableRow` | Staff member rows | `className='h-[52px]'` |
| `Badge` | Status: Active (green) / Deactivated (muted) | `variant='outline'` |
| `Badge` | Role: Dentist-Owner (navy) / Staff (default) | `variant='secondary'` |
| `Button` | "+ Add Staff" | `variant='default'` |
| `Button` | Edit (per row) | `variant='ghost'`, `size='icon'` |
| `Button` | Deactivate (per row) | `variant='ghost'`, `className='text-destructive'` |
| `Button` | Reactivate (per row, visible only on deactivated accounts) | `variant='ghost'` |
| `Input` | Staff name field (inline form) | `type='text'` |
| `Select` | Role selector (Staff — scheduling only / Staff — full operational) | — |
| `Input` | PIN field (exactly 6 digits) | `type='password'`, `maxLength={6}`, `minLength={6}` |
| `Alert` | Tier limit warning (Solo: 1 slot, Practice: 3 slots) | `variant='warning'` |
| `AlertDialog` | Deactivate confirmation | — |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Staff Name | text | Yes | Non-empty, max 100 chars | Empty | `Staff.name` |
| Role | select | Yes | Solo tier: "Staff — scheduling only" (only option). Practice tier: "Staff — scheduling only" or "Staff — full operational" | Staff — scheduling only | `Staff.role` |
| PIN | password | Yes | Exactly 6 digits, numeric only | Empty | `Staff.pin` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Staff Management" heading + staff count (e.g., "1 of 1 slots used"). Right: "+ Add Staff" primary button.
2. **Tier Info** — `px-6 py-2`. Subtle text: "Solo tier: 1 staff account" or "Practice tier: up to 3 staff accounts."
3. **Staff Table** — `px-6 py-4`. Columns: Name, Role (badge), Status (badge), Last Login (date), Actions (Edit, Deactivate/Reactivate). Dentist-owner's own row is always first (not editable, not deactivatable).
4. **Inline Add Form** — appears below table when "+ Add Staff" is tapped: Name Input + Role Select + PIN Input + Save/Cancel buttons. Collapses after save.
5. **Tier Limit Alert** — when all slots filled: Alert (warning): "All staff slots used. Upgrade to Practice tier for more." with "Learn More" link.

### Key Interactions

- **"+ Add Staff"** → inline form appears below table. Fill name, select role, set PIN. Save creates account. Cancel collapses form.
- **Edit** → row fields become editable inline. Name and Role editable. PIN shows "Change PIN" link that reveals PIN input. Save/Cancel appear.
- **Deactivate** → AlertDialog: "Deactivate [Staff Name]? They will no longer be able to log in." with Cancel + Deactivate (destructive). Deactivated accounts stay in the table with muted badge.
- **Reactivate** → immediate (no confirmation). Badge changes to Active. Staff can log in again.
- **Tier slot limits** → Solo tier: 1 staff slot. When filled, "+ Add Staff" is disabled with tooltip: "Solo tier supports 1 staff account." Practice tier: 3 slots. Same pattern when all 3 filled.
- **Dentist-owner row** → always visible at top of table. Cannot be edited or deactivated from this screen (managed in Settings).
- **Staff cannot access this screen** — role-gated. If a staff user navigates to `/staff`, they see: "Access restricted to Dentist-Owner."
- **Deactivated staff slot counting** → deactivated accounts do NOT count toward tier limits. A Solo-tier dentist with 1 deactivated staff can add a new staff member without reactivating the old one.
- **Staff PIN reset** → if a staff member is locked out (forgot PIN), the dentist-owner edits their account and sets a new PIN via "Change PIN." No self-service PIN recovery for staff.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Header renders. Table shows shimmer rows. |
| **Empty (no staff)** | Table empty. Message: "No staff accounts yet. Add your first staff member." + "+ Add Staff" button. |
| **All slots filled** | Alert banner below header. "+ Add Staff" button disabled. |
| **Staff role (access denied)** | Full screen message: "Access restricted to Dentist-Owner." Back to previous screen. |
| **Error** | Toast: "Failed to load staff accounts" + Retry. |

### CRUD Interactions

#### Adding (staff account)
- **Trigger:** "+ Add Staff" button
- **Opens:** Inline form below table
- **Form fields:** Staff Name, Role, PIN (see Field Specs)
- **Validation:** Name required. Role required. PIN must be exactly 6 digits.
- **On success:** Toast "Staff account created." New row appears in table. Form collapses.
- **On cancel:** Form collapses. No account created.
- **Blocked when:** All tier slots filled.

#### Editing
- **Trigger:** Edit button per row
- **Opens:** Inline — row fields become editable
- **Differences from Add:** PIN shows "Change PIN" link (optional). Name and Role editable.
- **On success:** Toast "Staff account updated." Row returns to read-only.

#### Deactivating
- **Trigger:** Deactivate button per row
- **Confirmation:** AlertDialog: "Deactivate [Staff Name]?" with Cancel + Deactivate (destructive)
- **On success:** Toast "Staff account deactivated." Badge changes to Deactivated (muted). Reactivate button appears.
- **Blocked when:** Cannot deactivate the Dentist-Owner account.

### What This Screen Does

- Lists all staff accounts with name, role, status, and last login
- Supports adding, editing, deactivating, and reactivating staff accounts
- Enforces tier-based slot limits (Solo: 1, Practice: 3)
- Assigns role-based permissions (scheduling-only vs full operational)
- Gates access to Dentist-Owner only

### What This Screen Does NOT Do

- Does NOT manage the dentist-owner's own account — that is Settings (Module 7)
- Does NOT support unlimited staff — Phase 2 (FR16.5)
- Does NOT offer fine-grained per-screen permission customization — predefined role templates only
- Does NOT manage multi-dentist profiles — Phase 2 (FR16)

### Wireframe

```
┌──────────────────────────────────────────────────┐
│ Staff Management          1 of 1 slots   [+Add] │
│ Solo tier: 1 staff account                       │
├──────────────────────────────────────────────────┤
│ Name          Role           Status   Last Login │
│──────────────────────────────────────────────────│
│ Dr. Owner     Dentist-Owner  Active   Today      │
│ Maria S.      Staff-Sched    Active   03/23/26   │
│──────────────────────────────────────────────────│
│ ⚠ All staff slots used. Upgrade for more.       │
└──────────────────────────────────────────────────┘
```
