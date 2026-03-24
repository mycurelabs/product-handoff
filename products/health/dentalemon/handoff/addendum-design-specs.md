---
type: addendum
scope: design-specs
product: dentalemon
module: cross-module
date: 2026-03-24
depends_on:
  - module-1-workspace/screen-dental-workspace.md
  - module-3-scheduling/screen-calendar.md
  - module-4-reporting/screen-analytics-dashboard.md
  - module-1-workspace/modal-consent-forms.md
  - module-7-settings/screen-settings.md
---

# Design Spec Addendum — Dentalemon

Closes 13 Critical implementation gaps identified in the Morgoth-readiness audit. This file is the authoritative source for animation values, component dimensions, and cross-module specs not covered in individual screen files.

---

## Zone 1: Timeline Carousel

The Timeline Carousel is Dentalemon's signature interaction — a Cover Flow + macOS Dock magnification hybrid. All values below are exact and final.

### 1.1 Spring Physics

| Property | Value | Notes |
|----------|-------|-------|
| `stiffness` | `300` | Custom "Carousel" preset — between Snappy (400) and Bouncy (300) |
| `damping` | `30` | High damping for healthcare context — no overshoot |
| `mass` | `1.2` | Heavier than default (1.0) for "gentle and unhurried" brand personality |
| `restDelta` | `0.01` | Stop animating when displacement < 0.01px |
| `restSpeed` | `0.01` | Stop animating when velocity < 0.01px/s |

```typescript
// Framer Motion transition object — carousel scroll
const carouselSpring = {
  type: "spring",
  stiffness: 300,
  damping: 30,
  mass: 1.2,
  restDelta: 0.01,
};
```

**Reduced-motion fallback:** Replace spring with `{ duration: 0.3, ease: "easeOut" }`. No bounce, immediate settle.

### 1.2 Dock Magnification Curve

The carousel uses proximity-based scaling: cards near the viewport center scale up, distant cards scale down.

| Property | Value |
|----------|-------|
| Proximity threshold | `200px` from viewport center |
| Min scale (distant) | `0.7` |
| Max scale (focal) | `1.0` |
| Interpolation | Cosine curve (smooth falloff, not linear) |

```typescript
// Framer Motion useTransform for Dock-style scaling
const distanceFromCenter = useMotionValue(0); // updated per frame during drag

const scale = useTransform(
  distanceFromCenter,
  [-400, -200, 0, 200, 400],  // input range (px from center)
  [0.7, 0.7, 1.0, 0.7, 0.7], // output range (scale)
  { ease: (t) => 0.5 * (1 + Math.cos(Math.PI * t)) } // cosine interpolation
);
```

**Reduced-motion fallback:** No scaling animation. Focal card = `scale: 1.0`, all others = `scale: 0.85` (static, no transition).

### 1.3 3D Transform Values

| Property | Value | Notes |
|----------|-------|-------|
| `perspective` | `1200px` | Subtle depth — 800px is too aggressive for healthcare |
| `rotateY` (focal card) | `0deg` | Faces directly at viewer |
| `rotateY` (adjacent ±1) | `±25deg` | Slight tilt toward center |
| `rotateY` (distant ±2+) | `±45deg` | Classic Cover Flow angle |
| `translateZ` (focal) | `0px` | At default depth |
| `translateZ` (non-focal) | `-100px` | Pushed back for parallax |

```typescript
const rotateY = useTransform(
  cardIndex, // relative to focal: -2, -1, 0, 1, 2
  [-2, -1, 0, 1, 2],
  [45, 25, 0, -25, -45] // degrees
);

const translateZ = useTransform(
  cardIndex,
  [-2, -1, 0, 1, 2],
  [-100, -50, 0, -50, -100] // px
);
```

**Container style:**
```css
.carousel-container {
  perspective: 1200px;
  perspective-origin: center center;
}
```

**Reduced-motion fallback:** No 3D transforms. Use horizontal offset only: focal card centered, adjacent cards offset by `cardWidth * 0.6` with `opacity: 0.6`. Flat 2D layout — no vestibular discomfort.

### 1.4 Card Dimensions

| Property | Value |
|----------|-------|
| Width (focal) | `320px` |
| Height (focal) | `240px` |
| Aspect ratio | `4:3` |
| Border radius | `16px` |
| Background | `#FFFFFF` (surface token) |
| Border | `1px solid #E5E5E5` |
| Shadow (focal) | `0 8px 32px rgba(0,0,0,0.12)` |
| Shadow (non-focal) | `0 4px 16px rgba(0,0,0,0.06)` |

**Card anatomy (top to bottom):**

| Zone | Height | Content |
|------|--------|---------|
| Date header | `36px` | Baseline date label ("Today's Baseline" or "Mar 15, 2026"), `px-4 py-2` |
| Chart preview | `168px` | Simplified per-quadrant dental chart render |
| Status bar | `36px` | Treatment count badge + session indicator, `px-4 py-2` |

**Non-focal cards:** Scale applies to entire card. At `scale: 0.7`, card renders as `224×168px`.

### 1.5 Card Spacing & Overlap

| Property | Value |
|----------|-------|
| Center-to-center offset | `180px` |
| Visible overlap | `140px` (320px card width - 180px offset) |
| Focal card z-index | `10` |
| Adjacent cards z-index | `5` |
| Distant cards z-index | `1` |
| Max visible cards | `5` (focal + 2 left + 2 right) |
| Cards beyond ±2 | `display: none` (performance — removed from DOM) |

### 1.6 Snap Behavior

| Property | Value |
|----------|-------|
| Snap target | Nearest card index by center position |
| Snap spring | `{ stiffness: 400, damping: 30, mass: 1 }` (Snappy preset — crisper than scroll) |
| Velocity threshold | `500px/s` — if release velocity exceeds this, snap to next card in drag direction |
| Deceleration | Not applicable — spring snap replaces deceleration |

```typescript
const onDragEnd = (event, info) => {
  const velocity = info.velocity.x;
  const offset = info.offset.x;

  if (Math.abs(velocity) > 500) {
    // Momentum snap: advance in drag direction
    setActiveIndex((prev) => prev + (velocity > 0 ? -1 : 1));
  } else if (Math.abs(offset) > cardCenterOffset / 2) {
    // Position snap: crossed halfway point
    setActiveIndex((prev) => prev + (offset > 0 ? -1 : 1));
  }
  // else: snap back to current card
};
```

**Reduced-motion fallback:** Immediate index change, no spring. `{ duration: 0 }`.

### 1.7 Drag Gesture Threshold

| Property | Value | Notes |
|----------|-------|-------|
| Drag threshold (horizontal) | `10px` | iOS standard — drag < 10px registers as tap |
| Drag axis | `"x"` | Horizontal only, no vertical drag |
| Drag elastic | `0.2` | Rubber-band at edges (first/last card) |
| Tap on tooth | Fires if drag distance < 10px | Opens slideout wizard for tapped tooth |

```typescript
<motion.div
  drag="x"
  dragConstraints={{ left: -maxScroll, right: 0 }}
  dragElastic={0.2}
  dragMomentum={false} // we handle snap manually
  onDragEnd={onDragEnd}
  // Tap handler only fires if drag < threshold
  onTap={(e) => handleToothTap(e)}
/>
```

### 1.8 activeIndex Contract

| Property | Value |
|----------|-------|
| Control model | **Controlled** — parent (DentalWorkspace) owns state |
| Prop | `activeIndex: number` |
| Callback | `onActiveIndexChange: (index: number) => void` |
| Initial value | Index of most recent baseline (today's if exists, else latest) |
| Sync targets | Breakdown table (filters to active baseline), Date picker (shows active date) |

```typescript
interface TimelineCarouselProps {
  baselines: Baseline[];
  activeIndex: number;
  onActiveIndexChange: (index: number) => void;
  onToothTap: (toothNumber: number) => void;
}
```

---

## Zone 6: Recharts ChartConfig

### 6.1 Dashboard Charts

**Line Chart — Revenue Trends**

```typescript
const revenueChartConfig: ChartConfig = {
  collections: {
    label: "Collections",
    color: "#FFCC5E", // primary amber
  },
  outstanding: {
    label: "Outstanding",
    color: "#EF4444", // error/destructive red
  },
};

// Y-axis formatter
const pesoFormatter = (value: number) =>
  `₱${value.toLocaleString("en-PH")}`;

// X-axis labels by period
const xAxisFormat = {
  weekly: "EEE",      // Mon, Tue, Wed...
  monthly: "MMM d",   // Jan 1, Jan 8...
  quarterly: "'Q'Q yyyy", // Q1 2026, Q2 2026...
};

// Tooltip
const tooltipFormatter = (value: number, name: string) =>
  [`₱${value.toLocaleString("en-PH")}`, name];
```

**Bar Chart — Top Treatments (Grouped)**

```typescript
const treatmentChartConfig: ChartConfig = {
  frequency: {
    label: "Procedures",
    color: "#1E3A5F", // navy
  },
  revenue: {
    label: "Revenue",
    color: "#FFCC5E", // primary amber
  },
};
```

| Property | Value |
|----------|-------|
| Orientation | Horizontal (treatments on Y-axis, values on X-axis) |
| Layout | **Grouped** (two bars per treatment, side by side) |
| Left X-axis | Count (procedures performed) |
| Right X-axis | Revenue (₱) |
| Bar gap | `4px` between grouped bars |
| Category gap | `16px` between treatment groups |
| Max treatments shown | `10` (top 10 by revenue, scrollable if more) |

> **Why grouped, not stacked:** Frequency (count) and revenue (₱) are different units. Stacking them in one bar is visually misleading — a ₱50,000 crown and a ₱500 cleaning would appear proportional to their price, not their count. Grouped bars let the viewer compare both dimensions independently.

**Pie Chart — Payment Methods**

```typescript
const paymentMethodConfig: ChartConfig = {
  cash: {
    label: "Cash",
    color: "#22C55E", // success green
  },
  creditCard: {
    label: "Credit Card",
    color: "#3B82F6", // info blue
  },
  bankTransfer: {
    label: "Bank Transfer",
    color: "#1E3A5F", // navy
  },
  other: {
    label: "Other",
    color: "#4A6B7A", // teal
  },
};
```

| Property | Value |
|----------|-------|
| Inner radius | `60%` (donut chart, not solid pie) |
| Label | Center label shows total: `₱{total}` |
| Legend | Bottom, horizontal, dot indicators |
| Tooltip | `{method}: ₱{value} ({percentage}%)` |

### 6.2 Bar Chart Decision: Grouped

Confirmed: **Grouped** layout with dual X-axes (see 6.1 above). Not stacked.

---

## Zone 7: Cross-Module Specs

### 7.1 E-Signature Canvas

Used in: Consent Forms Modal (Module 1), Settings Profile Tab (Module 7).

| Property | Value |
|----------|-------|
| Width | `100%` of parent container (minus padding) |
| Height | `100px` |
| Canvas resolution | `2x` (retina) — internal canvas is `width*2 × 200px` |
| Background | `#FFFFFF` (surface) |
| Border | `2px dashed #E5E5E5` |
| Border radius | `8px` (`rounded-lg`) |
| Stroke color | `#1E3A5F` (navy) |
| Stroke width | `2px` |
| Stroke cap | `round` |
| Min stroke length | `20px` total path length — rejects accidental dots/taps |
| Output format | PNG base64 at 2x resolution |
| Clear control | Ghost button "Clear" below canvas, right-aligned |
| Redo control | Ghost button "Undo" below canvas, left-aligned (removes last stroke) |

```typescript
interface SignatureCanvasProps {
  width: number;        // container width (px)
  height?: number;      // default 100
  strokeColor?: string; // default "#1E3A5F"
  strokeWidth?: number; // default 2
  onSignature: (base64: string | null) => void; // null when cleared
}
```

### 7.3 Calendar Slot Height

| Property | Value |
|----------|-------|
| Slot duration | `30 minutes` (configurable in Settings) |
| Slot height | `60px` |
| Time range | 7:00 AM – 7:00 PM (configurable in Settings) |
| Total slots | `24` (at default 30-min, 12-hour range) |
| Total grid height | `1440px` (24 × 60px) |
| Scroll behavior | Vertical scroll within the calendar content area |
| Initial scroll position | Scroll to current hour on load (8:00 AM = 120px offset) |
| Time label column | `60px` wide, sticky left, shows hour marks ("7 AM", "8 AM") |
| Half-hour divider | `1px dashed #E5E5E5` |
| Hour divider | `1px solid #E5E5E5` |

**Appointment block height formula:**
```
blockHeight = (durationMinutes / slotDuration) * slotHeight - gapPx
```

| Duration | Height | Notes |
|----------|--------|-------|
| 30 min | `56px` (60 - 4px gap) | Single slot |
| 60 min | `116px` (120 - 4px gap) | Two slots |
| 90 min | `176px` (180 - 4px gap) | Three slots |

**Week view compression:** At 7 columns in ~920px content area, each column is ~130px. Appointment blocks show:
- Patient first name only (truncated with ellipsis)
- Time range ("10:00–10:30")
- No procedure type label (hidden at compressed width)

**Month view overflow:** Each day cell shows first 2 patient names + "+N more" link. Tapping "+N more" switches to day view for that date.

---

## Framer Motion Token Reference

| Token | Duration | Use Case | Easing |
|-------|----------|----------|--------|
| `motion-micro` | `100–150ms` | Checkbox, toggle, hover feedback | `ease-out` |
| `motion-enter` | `200–300ms` | Dropdown open, tooltip appear, card mount | `ease-out` |
| `motion-page` | `250–350ms` | Route transition, modal enter/exit | `ease-in-out` |
| `motion-brand` | `300–500ms` | Carousel scroll, slideout resize, chart animate-in | `ease-in-out` or spring |

**Reduced-motion global pattern:**
```typescript
const prefersReducedMotion = usePrefersReducedMotion();

const transition = prefersReducedMotion
  ? { duration: 0 }
  : carouselSpring;
```

---

## Appendix: Cross-References

| Screen File | Addendum Section |
|-------------|-----------------|
| `module-1-workspace/screen-dental-workspace.md` | §1.1–1.8 (carousel) |
| `module-1-workspace/README.md` | §1.1–1.8 (Tech Notes) |
| `module-4-reporting/screen-analytics-dashboard.md` | §6.1–6.2 (charts) |
| `module-1-workspace/modal-consent-forms.md` | §7.1 (e-signature) |
| `module-3-scheduling/screen-calendar.md` | §7.2 (slot height) |
