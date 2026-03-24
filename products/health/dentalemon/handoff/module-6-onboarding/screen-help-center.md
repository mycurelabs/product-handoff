---
screen: Help Center
type: screen
route: /help
module: "Module 6: Onboarding"
tier: 3
components:
  - Input
  - Card
  - Badge
  - Accordion
  - AccordionItem
  - AccordionTrigger
  - AccordionContent
  - Button
  - Textarea
wireframe: inline-ascii
---

## Screen: Help Center

**Route:** `/help`
**POV:** A dentist or staff member looking for answers to questions about how to use Dentalemon, or reporting a bug.

### User Story

> As a dentist, I want to search a help center for answers to my questions so that I can solve problems without contacting support.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Report a Bug | Secondary | Opens inline bug report form | `outline` |
| Search | — | Filters articles by query | — |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Input` | Search field (top of page) | `type='search'`, `placeholder='Search help articles...'` |
| `Card` | Article cards (title + excerpt + category badge) | — |
| `Badge` | Category badges: Getting Started, Charting, Scheduling, Billing, Settings | `variant='outline'` |
| `Accordion` | FAQ sections (expandable question/answer pairs) | — |
| `AccordionItem` | Individual FAQ item | — |
| `AccordionTrigger` | Question text | — |
| `AccordionContent` | Answer text | — |
| `Button` | "Report a Bug" | `variant='outline'` |
| `Textarea` | Bug description field (inline form) | `placeholder='Describe the issue...'` |
| `Button` | "Submit Bug Report" | `variant='default'` |

### Field Specs

| Field | Type | Required | Validation | Default | Entity.Field |
|-------|------|----------|------------|---------|-------------|
| Search | text | No | Debounced 300ms | Empty | — |
| Bug Description | textarea | Yes (for bug report) | Non-empty, max 2000 chars | Empty | `BugReport.description` |

### Layout

1. **Header** — `px-6 py-4 flex justify-between items-center`. Left: "Help Center" heading. Right: "Report a Bug" outline button.
2. **Search** — `px-6 py-2`. Full-width search Input.
3. **Categories** — `px-6 py-4 flex gap-2`. Category filter pills: All, Getting Started, Charting, Scheduling, Billing, Settings.
4. **Article Grid** — `px-6 py-4 grid grid-cols-2 gap-4`. Article Cards with title, excerpt (2 lines), and category badge. Tapping a card expands to full article view (inline or navigates to detail).
5. **FAQ Section** — `px-6 py-4`. "Frequently Asked Questions" heading + Accordion with expandable Q&A items.
6. **Bug Report Form** — `px-6 py-4` (expanded inline when "Report a Bug" tapped). Bug description Textarea + device info (auto-captured) + "Submit Bug Report" primary + "Cancel" ghost.

### Key Interactions

- **Search** → debounced 300ms, full-text search across article titles and content. Results filter the article grid in real-time.
- **Category pills** → filter articles by category. Combinable with search.
- **Article card tap** → expands to full article view (inline accordion expand or separate detail view).
- **FAQ accordion** → tap question to expand/collapse answer.
- **"Report a Bug"** → expands inline form below header. Device info (model, OS version, app version) auto-captured. Dentist types description. "Submit Bug Report" saves locally + queues for sync. Toast: "Bug report submitted. We'll look into it."
- **Bug reports are free** — per PRD FR13.5, this is the hard line between free support and paid CRO. Bug reports always handled at no charge.
- **Offline** — help content bundled with the app. Search works offline. Bug reports queue locally and submit when connectivity is available.

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | N/A — content bundled with app, loads instantly. |
| **Empty (search, no results)** | "No articles match '[query]'" + "Clear search" link + "Report a Bug" suggestion. |
| **Error (content)** | N/A — local content, no network dependency. |
| **Error (bug submission)** | If local storage write fails during bug report submission: Toast "Failed to save bug report. Please try again." Form stays open with description preserved. |

### CRUD Interactions

#### Submitting (bug report)
- **Trigger:** "Report a Bug" → "Submit Bug Report"
- **Opens:** Inline form below header
- **Form fields:** Bug Description (textarea, required) + device info (auto-captured, read-only display)
- **Validation:** Description non-empty.
- **On success:** Toast "Bug report submitted." Form collapses. Report queued locally for sync.
- **On cancel:** Form collapses. No report submitted.

### What This Screen Does

- Provides searchable FAQs and help articles organized by category
- Supports in-app bug reporting with auto-captured device info
- Works fully offline (content bundled with app)
- Serves as the free support boundary (help center answers = no CRO charge)

### What This Screen Does NOT Do

- Does NOT provide live chat or real-time support
- Does NOT host tutorial videos — content creation is a marketing task (FR13.3 deferred)
- Does NOT manage CRO support tickets — those are operational
- Does NOT provide Dr. Lemon guided walkthrough tips — Phase 1 onboarding only

### Wireframe

```
┌──────────────────────────────────────────────────┐
│ Help Center                      [Report a Bug]  │
├──────────────────────────────────────────────────┤
│ [Search help articles...                       ] │
│                                                  │
│ All  Getting Started  Charting  Scheduling  ...  │
│                                                  │
│ ┌─────────────────┐  ┌─────────────────┐        │
│ │ How to add a    │  │ Setting up your │        │
│ │ new patient     │  │ fee schedule    │        │
│ │ Getting Started │  │ Getting Started │        │
│ └─────────────────┘  └─────────────────┘        │
│                                                  │
│ Frequently Asked Questions                       │
│ ▸ How do I export my data?                      │
│ ▸ Can I use Dentalemon offline?                 │
│ ▸ How do I change my PRC number?                │
└──────────────────────────────────────────────────┘
```
