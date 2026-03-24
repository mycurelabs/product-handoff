---
screen: Login
type: screen
route: /login
module: "Module 8: Auth"
tier: 3
components:
  - Card
  - Avatar
  - Badge
  - InputOTP
  - Button
  - Alert
wireframe: inline-ascii
---

## Screen: Login

**Route:** `/login`
**POV:** A dentist or staff member authenticating to access the app — selecting their account and entering their PIN.

### User Story

> As a dentist or staff member, I want to log in with a simple PIN so that I can access the app quickly between patients without typing long passwords.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| User card (tap) | — | Selects the user account for PIN entry | — |
| Login | Primary | Authenticates with entered PIN | `default` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Card` | User account cards (avatar + name + role badge) | `className='cursor-pointer'` |
| `Avatar` | User photo or initials | `size='lg'` |
| `Badge` | Role badge: Dentist-Owner / Staff | `variant='secondary'` |
| `InputOTP` | PIN entry field (exactly 6 digits) | `maxLength={6}`, `minLength={6}` |
| `Button` | "Login" submit | `variant='default'` |
| `Alert` | Lockout warning after failed attempts | `variant='destructive'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| User Selection | card select | Yes | Must tap a user card | Last-active user (on re-auth) | — |
| PIN | OTP input (6 digits) | Yes | Exactly 6 digits, numeric only. 5 failures → 30s lock. 10 failures → 5 min lock. | Empty | — |

### Layout

1. **App Logo** — centered at top. Dentalemon logo + tagline.
2. **User Cards** — `flex gap-4 justify-center`. One Card per user account (dentist-owner + active staff). Each card: Avatar (large) + Name + Role badge. Tap to select (selected card has amber border).
3. **PIN Entry** — `mt-6 flex justify-center`. InputOTP (6 digit circles). Appears after user card is selected.
4. **Login Button** — `mt-4 flex justify-center`. "Login" primary button (full-width, max-w-[300px]).
5. **Lockout Alert** — `mt-4`. Appears after 5 failed attempts: "Too many attempts. Try again in [N] seconds."

### Key Interactions

- **Tap user card** → card highlights with amber border. PIN entry field appears below. If re-auth after iPad sleep: last-active user pre-selected, PIN entry immediate.
- **Enter PIN** → each digit fills a circle. On complete (6 digits entered): auto-submit or tap "Login."
- **Successful login** → redirects to Patient List (`/patients`). If re-auth after iPad sleep: resumes exact previous screen with all state preserved (including open workspace, wizard state, etc.).
- **Failed PIN** → shake animation on PIN circles + "Incorrect PIN" text. Circles clear. Focus returns to first digit.
- **Lockout** → after 5 failures: Alert "Too many attempts. Try again in 30 seconds." PIN entry disabled. Countdown visible. After 10 total: 5 minute lockout.
- **First-time launch** → no user accounts exist in local database → auto-redirect to `/onboarding` (Module 6). Login screen never shown.
- **Single user (Solo tier, no staff)** → only one card shown (dentist-owner). Card auto-selected. PIN entry appears immediately.
- **Touch targets** → user cards and PIN digit circles ≥ 44x44px. Large enough for post-handwash damp fingers.
- **Dentist-owner PIN recovery** → if the dentist-owner forgets their PIN, the login screen shows a "Forgot PIN?" link after 3 failed attempts. This opens a recovery flow: answer the security question set during onboarding (e.g., "What is your mother's maiden name?"). On correct answer: PIN reset form appears (new PIN + confirm). On wrong answer: "Contact support for account recovery." Security question is set during Onboarding Step 2 (Profile) — add to Module 6 Field Specs.
- **Inactivity timeout** → if the app is open but idle for a configurable period (default: 5 minutes), the session auto-locks and the login screen appears. Timeout configurable in Settings (Module 7). Same re-auth flow as iPad sleep.
- **Lockout ceiling** → after 10 failed attempts, lockout remains at 5 minutes per subsequent 5-attempt block. No permanent lockout — this is a local device with no remote wipe capability.
- **State preservation scope** → on re-auth (iPad sleep or inactivity timeout), the following state is preserved: current route, scroll positions, form field values (unsaved), open modals/sheets/slideouts, wizard step + selections, carousel position. Payment flows in progress are preserved but payment submission is NOT re-attempted — the user must tap "Record" again after re-auth.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Logo renders. User cards show skeleton placeholders briefly. |
| **Single user** | One card shown, auto-selected. PIN entry appears immediately. |
| **Multiple users** | Cards shown side-by-side. PIN entry hidden until a card is tapped. |
| **Failed attempt** | Shake animation + "Incorrect PIN." Circles clear. |
| **Locked out (5 attempts)** | Destructive Alert with countdown. PIN disabled. |
| **Locked out (10 attempts)** | 5-minute lockout. Same Alert pattern with longer countdown. |
| **First-time (no accounts)** | Auto-redirect to `/onboarding`. Login screen not shown. |
| **Re-auth (iPad wake)** | Last-active user pre-selected. PIN entry immediate. After success: resume previous screen. |
| **Empty** | N/A — at least one account (dentist-owner) always exists after onboarding. First-time state handled by redirect to `/onboarding`. |
| **Error** | N/A — all authentication data is local. If local storage is corrupted: full-screen message "Data error. Please reinstall or contact support." |

### Navigation

| CTA | Destination | Route | Module |
|-----|-------------|-------|--------|
| Successful login | Patient List | `/patients` | Module 2: Patient Management |
| First-time redirect | Onboarding Wizard | `/onboarding` | Module 6: Onboarding |
| Re-auth success | Previous screen (any) | (preserved route) | Any module |

### What This Screen Does

- Authenticates the dentist-owner and staff via PIN
- Supports multi-user selection (dentist + staff accounts)
- Handles auto-lock/re-auth after iPad sleep with state preservation
- Enforces brute-force protection (lockout after failed attempts)
- Redirects first-time users to onboarding

### What This Screen Does NOT Do

- Does NOT create accounts — dentist account created in Onboarding (M6), staff in Staff Management (M5)
- Does NOT support biometric auth (Face ID) — Phase 2
- Does NOT connect to cloud auth — local PIN validation only
- Does NOT support email-based password reset — dentist-owner uses security question recovery; staff PINs reset by owner in M5

### Wireframe

```
┌──────────────────────────────────────┐
│                                      │
│          🍋 Dentalemon               │
│     "Because you do it all."         │
│                                      │
│    ┌──────────┐  ┌──────────┐       │
│    │  👤      │  │  👤      │       │
│    │ Dr. Aves │  │ Maria S. │       │
│    │  Owner   │  │  Staff   │       │
│    └──────────┘  └──────────┘       │
│                                      │
│        ○ ○ ○ ○ ○ ○                  │
│         Enter PIN                    │
│                                      │
│        [    Login    ]               │
│                                      │
└──────────────────────────────────────┘
```
