# Phase 1: Inputs & Canvas Placeholder

**Goal:** Build the complete control panel and canvas placeholder. All inputs wired to reactive state. No spirograph drawing logic yet — just the shell.

**Tech Stack:** Svelte 5 / SvelteKit, shadcn-svelte components, Tailwind CSS, HTML5 `<canvas>` placeholder.

---

## 1. State Model

A single reactive object (`$state` rune) named `spiroConfig`:

| Key             | Type                                | Default          | Description                                                                                                              |
| --------------- | ----------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `mode`          | `"prerender" \| "auto" \| "manual"` | `"prerender"`    | **Prerender**: instant static curve. **Auto**: animated draw. **Manual**: user steps through each frame via Next button. |
| `variant`       | `"hypotrochoid" \| "epitrochoid"`   | `"hypotrochoid"` | Hypotrochoid = wheel rolls inside stator. Epitrochoid = wheel rolls outside stator.                                      |
| `statorTeeth`   | `number`                            | `96`             | Outer ring gear teeth count.                                                                                             |
| `rotatorTeeth`  | `number`                            | `36`             | Inner/outer wheel gear teeth count.                                                                                      |
| `penPosition`   | `number` (0–120)                    | `50`             | Distance from rotator center as **% of rotator radius**. >100 allowed for outside-the-wheel effects.                     |
| `penColor`      | `string` (hex)                      | `"#3b82f6"`      | Stroke color.                                                                                                            |
| `lineWidth`     | `number` (0.5–5)                    | `1`              | Stroke thickness in px.                                                                                                  |
| `showMechanism` | `boolean`                           | `false`          | Toggle faint stator ring + rotator wheel + pen arm overlay.                                                              |
| `isDrawing`     | `boolean`                           | `false`          | Runtime flag — true while auto/manual is actively drawing.                                                               |
| `progress`      | `number` (0–1)                      | `0`              | Normalized draw progress. Manual mode uses this directly.                                                                |

**Derived Values (computed):**

- `statorRadius` = `statorTeeth * toothSize` (`toothSize` fixed constant, e.g. `2px`)
- `rotatorRadius` = `rotatorTeeth * toothSize`

> Background is always white. No opacity control.

---

## 2. UI Layout

### 2.1 Overall Page Structure

**Desktop (≥768px):**

```
├──────────────────────┬───────────────────────┤
│                      │                       │
│   Control Panel      │     Canvas Area       │
│   (left sidebar)     │     (center/right)    │
│   ~280px width       │     flexes to fill    │
│                      │                       │
└──────────────────────┴───────────────────────┘
```

**Mobile (<768px):**

```
┌──────────────────────────────────────────────┐
│  Canvas Area (top, full-width)               │
│  Square aspect ratio, ~90vw max              │
├──────────────────────────────────────────────┤
│  Control Panel (bottom, scrollable)          │
│  Collapsible sections or accordion           │
└──────────────────────────────────────────────┐
```

- **Desktop:** Sidebar on the left, canvas on the right. Sidebar scrolls independently if content overflows.
- **Mobile:** Canvas stacks on top. Controls below in a scrollable panel. Sections can be collapsible (accordion) to reduce vertical scroll.
- **Canvas size:** Responsive square. Desktop: `min(500px, 90vw)`. Mobile: `90vw`. Internal resolution = CSS size × `devicePixelRatio`.

### 2.2 Control Panel Sections

#### A. Mode & Playback

- **Mode selector:** Segmented control (Prerender / Auto / Manual).
- **Action buttons:** `Draw / Redraw` (primary), `Clear`.
- `Pause / Resume` — shown only in Auto mode.
- `Next Step` — shown only in Manual mode.
- **Progress bar** — shown in Auto/Manual.

#### B. Geometry

- **Variant:** Segmented control (Hypotrochoid / Epitrochoid).
- **Stator Teeth:** Number input `min=10 max=200 step=1`. Shows derived radius read-only (e.g., "192px").
- **Rotator Teeth:** Number input. Constraint: `rotatorTeeth < statorTeeth` when Hypotrochoid.
- **Pen Position (%):** Range slider `0–120` with number input beside it.
- **Show Mechanism:** Toggle switch.

#### C. Appearance

- **Pen Color:** Native color picker (compact).
- **Line Width:** Range slider `0.5–5` step `0.5`.

#### D. Presets & Discovery

- **Preset selector:** Dropdown (Classic Flower, Tight Rings, Star Burst, Simple Circle).
- **Randomize** button — randomizes teeth + pen, keeps variant.
- **Reset** button — resets to default config.

---

## 3. Canvas Placeholder

- Centered `<canvas>`, white background always.
- **Phase 1 behavior:**
  - Faint dotted grid or crosshair at center.
  - Ghosted circle for `statorRadius` — updates live with teeth changes.
  - If `showMechanism` true: ghosted `rotatorRadius` circle + pen arm at fixed angle (e.g., 0°).
  - Idle text overlay: "Press Draw to generate".
- **Responsive:** CSS scales the canvas; JS sets `width`/`height` × `devicePixelRatio` for crisp rendering.

---

## 4. Interactions & State Wiring

| User Action               | Phase 1 Behavior                                                                                                                                      |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Change any input          | State updates. Placeholder redraws (ghost circles). No curve.                                                                                         |
| Click **Draw**            | Prerender: clears canvas, shows placeholder mock shape (circle/static path) to verify pipeline. Auto/Manual: sets `isDrawing = true`, `progress = 0`. |
| Click **Clear**           | Clears to white. Resets `progress = 0`, `isDrawing = false`.                                                                                          |
| Click **Randomize**       | Randomizes teeth + pen. Ghost circles update.                                                                                                         |
| Select **Preset**         | Loads config. Ghost circles update.                                                                                                                   |
| Toggle **Show Mechanism** | Ghost overlay on/off.                                                                                                                                 |

> No actual spirograph math in Phase 1. Draw button renders a temporary stand-in shape to verify canvas + color + lineWidth pipeline.

---

## 5. Validation & Constraints

- `statorTeeth > rotatorTeeth` enforced when variant is `hypotrochoid`. UI shows error state if violated.
- All numeric inputs clamped to min/max.
- Pen position >100% allowed; subtle hint that this is "extended arm" behavior.

---

## 6. File Structure

```
src/
├── routes/
│   └── +page.svelte              # Main layout, responsive sidebar/canvas
├── lib/
│   ├── stores/
│   │   └── spiroConfig.svelte.ts  # $state object + derived radius
│   ├── components/
│   │   ├── ControlPanel.svelte   # All input sections
│   │   ├── SpiroCanvas.svelte    # Canvas + placeholder drawing
│   │   ├── ModeSelector.svelte   # Segmented control
│   │   ├── TeethInput.svelte     # Number input + derived radius
│   │   ├── PenPositionInput.svelte # Slider + %
│   │   ├── PresetSelector.svelte # Dropdown + randomize + reset
│   │   └── PlaybackControls.svelte # Draw / Clear / Pause / Next
│   └── utils/
│       └── presets.ts            # Preset config objects
```

---

## 7. Success Criteria

- [ ] All inputs exist and are styled with shadcn components.
- [ ] State updates reactively update derived values and ghost circles.
- [ ] Canvas renders grid + ghost stator circle + optional mechanism overlay.
- [ ] Mode switching shows/hides relevant controls.
- [ ] Validation prevents invalid hypotrochoid combinations.
- [ ] Presets, randomize, reset all work.
- [ ] Draw button triggers placeholder render with correct color + lineWidth.
- [ ] Layout is responsive: sidebar+canvas on desktop, stacked on mobile.
- [ ] Mobile controls are usable (no overflow, reasonable tap targets ≥44px).
