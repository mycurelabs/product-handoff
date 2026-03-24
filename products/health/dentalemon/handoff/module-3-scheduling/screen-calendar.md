---
screen: Calendar
type: screen
route: /calendar
module: "Module 3: Scheduling"
tier: 2
components:
  - Calendar
  - Tabs
  - TabsTrigger
  - Button
  - Card
  - Badge
  - Popover
  - Dialog
  - AlertDialog
wireframe: wireframes/screen-calendar.xml
wireframe_pages: [Default, Empty, Error]
  - Default
  - Empty
  - Error
---

## Screen: Calendar

**Route:** `/calendar`
**POV:** A dentist or staff member managing the day's appointments — viewing scheduled patients, checking in arrivals, handling walk-ins, and planning the week ahead.

### User Story

> As a dentist, I want a native calendar with day, week, and month views so that I can manage my schedule without relying on Google Calendar or external tools.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| + New Appointment | Primary | Opens New Appointment modal | `default` |
| Walk-In | Secondary | Routes to Patient List with registration modal pre-opened for walk-in flow | `outline` |
| Day / Week / Month | — | View toggle (segmented control) | — |
| Today | Ghost | Jumps calendar to today's date | `ghost` |
| ← / → | Ghost | Navigate to previous/next day/week/month | `ghost` |
| Appointment block (tap) | — | Opens appointment detail popover | — |
| Check In (on appointment) | Primary | Marks appointment as Checked In → opens workspace | `default` |
| Edit (on appointment) | Secondary | Opens New Appointment modal pre-filled with all fields for in-place update | `outline` |
| Revert No-Show (on No-Show appointment) | Ghost | Restores appointment from No-Show to Scheduled status without opening workspace | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Calendar` | Month view mini-calendar (sidebar or inline) for date navigation | — |
| `Tabs` | Day / Week / Month view toggle | `defaultValue='day'` |
| `TabsTrigger` | Day view trigger | `value='day'` |
| `TabsTrigger` | Week view trigger | `value='week'` |
| `TabsTrigger` | Month view trigger | `value='month'` |
| `Button` | "+ New Appointment" CTA | `variant='default'` |
| `Button` | "Walk-In" CTA | `variant='outline'` |
| `Button` | "Today" quick jump | `variant='ghost'` |
| `Button` | ← / → nav arrows | `variant='ghost'`, `size='icon'` |
| `Card` | Appointment block within time slots (patient name, time, procedure type, status badge) | — |
| `Badge` | Appointment status: Scheduled (default), Checked In (blue), Completed (green), Cancelled (muted), No-Show (red) | `variant='outline'` |
| `Popover` | Appointment detail popover (on tap) | — |
| `Button` | Check In (inside popover) | `variant='default'` |
| `Button` | Reschedule (inside popover) | `variant='outline'` |
| `Button` | Cancel (inside popover) | `variant='destructive'` |
| `Button` | Mark No-Show (inside popover) | `variant='ghost'` |
| `Button` | Edit (inside popover) | `variant='outline'` |
| `Button` | Revert No-Show (inside popover, shown only on No-Show appointments) | `variant='ghost'` |
| `Dialog` | New Appointment modal container | — |
| `AlertDialog` | Cancel appointment confirmation | — |

### Field Specs

> N/A — Calendar is a read-only display. All input fields (patient, date, time, procedure) are in the New Appointment modal (see next comment).

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Calendar" heading + date range label (e.g., "March 24, 2026" for day, "Mar 24-30" for week). Right: "Walk-In" outline + "+ New Appointment" primary.
2. **View Controls** — `px-6 py-2 flex justify-between items-center`. Left: ← Today → navigation. Right: Day / Week / Month segmented control (Tabs).
3. **Calendar Body** — varies by view:
   - **Day View** — `px-6`. Vertical time grid (7:00 AM → 7:00 PM, configurable). Each slot = 30 min (configurable), `60px` height per slot. See [`addendum-design-specs.md`](../../addendum-design-specs.md) §7.2 for slot dimensions, scroll behavior, and appointment block height formula. Appointment Cards positioned in their time slots (height proportional to duration). Tapping empty slot → New Appointment modal pre-filled with that time.
   - **Week View** — `px-6`. 7-column grid (Mon-Sun). Each column = day view compressed. Appointment Cards as colored blocks. Tapping a block → detail popover.
   - **Month View** — `px-6`. Standard month grid. Each day cell shows appointment count badge + first 2 patient names. Tapping a day → switches to Day View for that date.
4. **No appointments indicator** — slots without appointments show as empty time blocks. No visual clutter.

### Key Interactions

- **Tap empty time slot (day view)** → opens New Appointment modal pre-filled with the tapped date + time.
- **Tap appointment block** → Popover with: patient name + photo, time, procedure type, status badge, and actions: Check In (primary), Reschedule, Cancel (destructive), Mark No-Show.
- **Check In** → marks appointment as "Checked In" (blue badge). Opens Dental Workspace (`/workspace/:patientId`) for that patient. Full-screen takeover.
- **Reschedule** → opens New Appointment modal pre-filled with patient + procedure, but with date/time editable. **Atomicity:** the original appointment is only cancelled AFTER the new appointment is successfully created. If creation fails, the original stays intact.
- **Cancel** → AlertDialog: "Cancel [Patient Name]'s appointment on [date] at [time]?" with Keep + Cancel Appointment (destructive). On cancel: time slot freed, appointment marked "Cancelled" (muted badge, stays visible for the day).
- **Mark No-Show** → marks appointment as "No-Show" (red badge). No confirmation needed. **Revert:** on a No-Show appointment, the popover shows "Revert No-Show" instead of "Mark No-Show." Tapping it restores the "Scheduled" status without opening the workspace (unlike Check In).
- **Edit** → opens New Appointment modal pre-filled with ALL fields (patient, date, time, duration, procedure, notes) editable. Unlike Reschedule, Edit does NOT cancel the original — it updates in place. Use for correcting procedure type, notes, or duration without changing the time slot.
- **Walk-In** → routes to Patient List (`/patients`) with the New Patient Registration modal pre-opened and a "Walk-in" context banner. **Return flow:** after patient creation, the system auto-redirects back to the Calendar with the New Appointment modal pre-opened, patient pre-selected, time set to "now" or next available slot, date locked to today. If staff abandons the registration (cancels the modal or navigates away from Patient List), they return to the Calendar manually — no appointment is created.
- **"+ New Appointment"** → opens New Appointment modal with today's date, no time pre-selected.
- **"Today" button** → jumps calendar to today's date in the current view mode.
- **← / → arrows** → navigate backward/forward by day (day view), week (week view), or month (month view).
- **Day/Week/Month toggle** → switches view. Current date context preserved.
- **Double-booking visual** → when two appointments overlap in the same time slot, both blocks are shown side-by-side (split the slot width 50/50) with an amber border indicating the overlap.
- **Past appointments** → visible but muted. Check In action hidden. Reschedule still available.
- **Completed appointments** → green badge. Patient returned from workspace. Visible for the day, then only in week/month summary counts.

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Check In | Dental Workspace | `/workspace/:patientId` | Module 1: Dental Workspace |
| Walk-In | Patient List (registration modal) | `/patients` | Module 2: Patient Management |

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Header + view controls render. Calendar body shows shimmer blocks in time slots. |
| **Empty (no appointments today)** | Time grid renders empty. Centered message in the body: "No appointments today." + "+ New Appointment" button. |
| **Empty (no appointments this week/month)** | Grid renders. "No appointments this [week/month]." in the body area. |
| **Error** | Toast: "Failed to load appointments" + Retry. Last known calendar state remains visible. |

### CRUD Interactions

#### Adding (appointment — via modal, see next comment)
- **Trigger:** Tap empty time slot OR "+ New Appointment" button
- **Opens:** New Appointment modal (see Modal comment)

#### Rescheduling
- **Trigger:** Appointment popover → Reschedule
- **Opens:** New Appointment modal pre-filled with patient + procedure, date/time editable
- **On success:** Original appointment cancelled, new appointment created at new time. Toast: "Appointment rescheduled to [new date/time]."
- **On cancel:** Popover closes, no change.

#### Cancelling
- **Trigger:** Appointment popover → Cancel
- **Confirmation:** AlertDialog: "Cancel [Patient Name]'s appointment?" with Keep + Cancel Appointment (destructive)
- **Soft vs hard delete:** Soft cancel — appointment stays visible for the day with "Cancelled" badge, removed from future views.
- **On success:** Toast: "Appointment cancelled." Time slot freed. Badge changes to Cancelled (muted).

#### Editing
- **Trigger:** Appointment popover → Edit
- **Opens:** New Appointment modal pre-filled with ALL fields (patient, date, time, duration, procedure, notes). All fields editable.
- **Differences from Create:** Submit button shows "Save Changes" instead of "Create Appointment." No cancellation of original — updates in place.
- **On success:** Toast: "Appointment updated." Calendar block reflects changes. Popover closes.
- **On cancel:** Modal closes, no change.

#### Marking No-Show
- **Trigger:** Appointment popover → Mark No-Show
- **Confirmation:** None — low-risk, reversible action.
- **On success:** Badge changes to No-Show (red). Reversible via "Revert No-Show" button in the popover (restores to Scheduled status without opening workspace).

#### Reverting No-Show
- **Trigger:** Appointment popover → Revert No-Show (visible only on No-Show appointments, replaces "Mark No-Show")
- **Confirmation:** None — restores to previous state.
- **On success:** Badge changes back to Scheduled (default). "Mark No-Show" button reappears in popover.

### What This Screen Does

- Displays appointments in day/week/month calendar views
- Supports appointment creation via time slot tap or "+ New Appointment"
- Provides appointment check-in that opens the workspace
- Handles rescheduling, cancellation, and no-show marking
- Shows double-booking conflicts visually (side-by-side blocks)
- Supports walk-in flow (routes to Patient List for registration + adds to today's schedule)
- Navigates between dates with Today jump + ← / → arrows

### What This Screen Does NOT Do

- Does NOT send appointment reminders — Phase 2 (FR19)
- Does NOT support patient self-booking — not in Phase 1
- Does NOT show per-dentist calendars — Phase 2 (FR16, multi-dentist)
- Does NOT sync with Google Calendar or any external service — never (FR6.1)
- Does NOT manage chair assignments — Phase 2
- Does NOT display inside the workspace — scheduling is outside the workspace shell
