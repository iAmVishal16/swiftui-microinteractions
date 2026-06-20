---
name: swiftui-microinteractions
description: Generate premium SwiftUI animations in the legendary-Animo style — spring physics, CoreHaptics, glass morphism, SF Symbols 7 draw animations, complete compilable files.
---

Generate a complete, compilable SwiftUI animation file in the legendary-Animo style. No placeholders. No TODO comments.

## When to Use
- Building a SwiftUI button, gesture, slider, or liquid effect with premium physics
- Need haptic feedback timed correctly to animation phases
- Creating a drag interaction with resistance, snap, or threshold trigger
- Animating SF Symbols with draw-on, breathe, bounce, replace, or variable color effects (iOS 17–26)
- Building a Canvas loader that traces a shape outline with a comet trail (infinity, star, polygon…)
- Editing an existing SwiftUI animation file

## Mode Detection
- **Edit mode**: prompt contains "edit", "update", "add X to", "change", or a `.swift` filename → read file first, change only what's asked, overwrite file on disk
- **Create mode**: everything else → generate a new complete file and write it to disk

---

## Intent Inference (runs before code generation)

Before generating any code, parse the prompt for under-specified details and resolve them to defaults. Print a `🔍 Inferred from prompt:` block so the user can see — and override — what was auto-resolved. Only print lines that were actually inferred (not ones the user explicitly stated).

### Step 0 — Normalize designer vocabulary

Translate any designer words in the prompt into skill-understood terms before running subsequent steps. This lets non-technical prompts ("snappy card that rips with anticipation") produce the same output as fully specified ones.

**Animation feel words → physics preset:**

| Designer says | Maps to |
|---|---|
| snappy / crisp / tight | UI pop `0.35/0.6` |
| bouncy / springy / elastic | Snap `0.35/0.5` |
| stretchy / rubber / wobbly | 3-phase rubber-band (see iOS 26 section) |
| melts / floats / soft / gentle | Slow morph `0.6/0.8` |
| stiff / precise / mechanical | Precision stiff `interpolatingSpring(220, 22)` |
| linear / steady / constant | `.linear(duration:)` |

**Motion principle words → archetype / animation signals:**

| Designer says | Implies |
|---|---|
| anticipation | pre-stretch phase before main move |
| follow-through | add overshoot to settle animation |
| squash & stretch | scale deformation on impact (x expand, y compress) |
| staging / stagger | offset `.animation(delay:)` per element |
| pulse / breathe | `.breathe` isActive loop |
| ripple | radial spread `Circle` scale from tap point |
| flood | Liquid Toggle archetype, metaball fill |
| wiggle / shake | `.wiggle value:` — error / rejection signal |
| pop | quick `.scaleEffect` overshoot + snap back |
| morph | Glass Morph archetype or shape interpolation |

**UI element names → SwiftUI primitives:**

| Designer says | Also called | SwiftUI |
|---|---|---|
| bottom sheet / tray / drawer / panel | pull-up, modal sheet | `.sheet` + `.presentationDetents` |
| modal / dialog / overlay / lightbox | full-screen sheet | `.sheet` (full) or `.fullScreenCover` |
| popover / tooltip / callout / bubble | hover card | `.popover` |
| toast / snackbar / nudge / banner | inline notification | custom overlay + auto-dismiss `.task` |
| alert / warning / confirm | dialog | `.alert` or `.confirmationDialog` |
| card / tile / surface | panel | `RoundedRectangle` + material |
| chip / tag / pill / badge | filter tag | custom `Capsule` |
| FAB / floating action button | floating button | absolute-positioned `Button` in `ZStack` |
| segmented control / pill selector / tab strip | toggle group | `Picker(.segmented)` or custom pill |
| skeleton / shimmer / placeholder | loading placeholder | `redacted(.placeholder)` or animated opacity |
| spinner / loader / throbber | activity indicator | `ProgressView` |
| contextual menu / long-press menu | press-hold menu | `.contextMenu` |
| action sheet / bottom action menu | options sheet | `.confirmationDialog` |

### Step 1 — Symbol → Effect lookup

If the prompt names a symbol (or describes one by subject) and does **not** specify an animation effect, apply this table:

| Symbol / subject | Type | Auto-effect |
|---|---|---|
| `signature`, `pencil`, `pencil.line`, `scribble`, `lasso` | stroke | `.drawOn` auto-loop |
| `checkmark`, `heart` (outline), `star` (outline), `wifi`, `antenna.radiowaves` | stroke | `.drawOn` auto-loop |
| `heart.fill`, `star.fill`, `circle.fill` | flood-fill | `.bounce value:` (drawOn invisible on fill shapes) |
| `circle.dotted`, `rays`, `waveform` | animated structure | `.variableColor.iterative.reversing` or `.breathe` |
| `arrow.clockwise`, `arrow.2.circlepath`, `gear` | rotation | `.rotate` (indefinite, `isActive`) |
| `bell`, `bell.fill` | discrete action | `.bounce value:` |

### Step 2 — Showcase mode vs. interaction mode

- **No user action described** (no "tap to", "on submit", "when user"…) → **showcase mode**: drive with `.task` auto-loop. Symbol animates on its own without any tap required.
- **User action described** → gate the animation behind that action only. Do not add an auto-loop.

### Step 3 — Apply Defaults Contract

Resolve any unspecified UI elements using the Defaults Contract (see below).

### Step 4 — Print inference log

```
🔍  Inferred from prompt:
    Vocab     → "<designer word>" → <normalized term>  (only if Step 0 fired)
    Symbol    → <name> (<stroke|fill>) → <effect>
    Mode      → showcase (auto-loop) | interaction (tap/submit)
    Container → .sheet (Apple default) | inline | none
    Controls  → Clear · Done | none
```

---

## Defaults Contract

**Absent = Apple HIG default.** Any UI element not mentioned in the prompt defaults to the Apple system primitive — never invent a custom modal, overlay, or container when a system one fits.

| Element | Apple default | When to escalate |
|---|---|---|
| Container | `.sheet(isPresented:)` | Prompt names custom modal / full-screen |
| Dismiss | `Button("Done")` in `.toolbar(.confirmationAction)` | Prompt names custom button |
| Clear / reset | `Button("Clear")` in `.toolbar(.cancellationAction)` | Drawing/signature context only |
| Background inside sheet | System sheet bg (no custom dark fill) | Prompt explicitly names dark/custom bg |
| Navigation | `NavigationStack` | Multi-step flow or title implied |
| Loading state | `.progressView()` or symbol `.variableColor` | Prompt names a specific loader shape |

**Drawing / signature context rule** — if the prompt contains "signature", "draw", "sketch", or "handwrite", automatically apply:
- Container: `.sheet(isPresented:)`
- Toolbar: `Button("Clear")` (`.cancellationAction`) + `Button("Done")` (`.confirmationAction`)
- No custom dark background inside the sheet (let the system sheet surface show through)

---

## Asset Scanning (run before generating any image-dependent animation)

If the animation involves images, photos, or cards, scan for existing assets first:

1. Check these paths in order:
   - `legendary-Animo/Res/Assets.xcassets/Images/`
   - `Assets.xcassets/Images/`
   - `Assets.xcassets/`

2. List `.imageset` folder names — strip `.imageset` suffix to get the `Image("name")` string

3. **If found** → use them directly. Print:
   ```
   🖼️  Assets found: photo1, photo2 … (using in file)
   ```

4. **If not found** → use `Image(systemName: "photo.fill")` in a colored `RoundedRectangle`. Include Asset Notes at the end.

Never invent asset names. Only reference names confirmed to exist on disk.

---

## Haptics — check project first

Before including haptic code, check if `HapticFeedback.swift` exists anywhere in the project:
- **If found** → call `HapticFeedback.lightImpact()` etc. directly. Do NOT re-declare the struct.
- **If not found** → call `UIImpactFeedbackGenerator` / `UISelectionFeedbackGenerator` directly inline, and add `import UIKit` at the top.

### When to add haptics vs. skip

**Add haptics when:**
- A gesture has a threshold that triggers an irreversible action (drag-to-delete, swipe-to-dismiss) → `heavyImpact` at the threshold crossing
- A toggle switches between two meaningful states → `mediumImpact` on commit
- A destructive action (delete, clear, reset) is confirmed → `heavyImpact`
- A scrub or dial moves across discrete data points → `selectionChanged` per step
- A glass morph or metaball fuse/separate completes → `mediumImpact`

**Skip haptics when:**
- Pure symbol showcase with auto-loop and no user interaction
- Loading indicator or ambient background animation (stars, particles drifting)
- Prompt describes no user action at all

**Haptic ladder for gesture arcs:**
1. `lightImpact` — drag starts / touch down
2. `mediumImpact` — threshold crossed / halfway
3. `heavyImpact` — action commits / destruction confirmed
4. `selectionChanged` — discrete value scrub / step

**SourceKit `HapticFeedback` false positive — always ignore:**
In files written to `Carousels/` or `Animations/`, SourceKit reports `Cannot find 'HapticFeedback' in scope`. This is **not a real error** — SourceKit analyzes the new file in isolation and doesn't see other module members. `HapticFeedback.swift` is registered in the Sources build phase; the real compiler resolves it correctly. Do not add `import UIKit`, do not re-declare the struct, do not alter the code.

---

## Style Rules

**Backgrounds:** `Color(white: 0.06).ignoresSafeArea()` standalone · `Color("BgColor").ignoresSafeArea()` in-project

**Glass surfaces (pre-iOS 26):** `.ultraThinMaterial` or `.thickMaterial` clipped to shape · track background `.white.opacity(0.06)` + 1pt stroke `.white.opacity(0.08)`

**Opacity levels:** ghost `0.06–0.08` · subtle `0.12` · inactive `0.3` · secondary `0.5` · active `0.8–0.9` · full `1.0`

**Colors:** white + opacity dominant · cyan `Color(red: 0.45, green: 0.65, blue: 1.0)` · green `Color(red: 0.55, green: 0.95, blue: 0.75)` · never `.blue/.green/.red` on dark bg

**Gradients:** two-tone only · blob fill `[.white, Color(white: 0.88)]` · progress `[cyan, green]`

---

## Light Theme

The Style Rules above are dark-first. For a **light** UI (dashboards, cards, sheets on white), depth comes from **tonal deltas, not shadows** — premium light design avoids drop shadows between stacked surfaces.

**Tonal stack — separate surfaces by lightness:**

| Role | Value |
|---|---|
| Screen background | `Color(white: 0.92)` (or gradient `0.93 → 0.87`) |
| Card / row surface | `Color.white` |
| Inset well (icon circle, field) | `Color(white: 0.95)` |
| Track / divider | `Color(white: 0.88)` |
| Ink (primary text) | `Color(white: 0.10)` |
| Muted (secondary text) | `Color(white: 0.55)` |

Rules:
- **No `.shadow` between stacked light surfaces** — a white row on a `0.92` background already reads as raised. Reserve a shadow for a single floating primary (e.g. a dark CTA), never for every row.
- Accent colors must be **deepened** for contrast on white — e.g. cyan `Color(red: 0.30, green: 0.52, blue: 0.95)`, the opposite of the dark-bg cyan.
- A dark capsule CTA (`Color(white: 0.10)` fill, white label) is the light-theme equivalent of the glass button.

### Adaptive (support BOTH light + dark)

When a screen must follow the system appearance, never hardcode `.white`/`.black` or force `.preferredColorScheme`. Drive everything off `@Environment(\.colorScheme)` + semantic colors:

```swift
@Environment(\.colorScheme) private var scheme
private var isDark: Bool { scheme == .dark }
```
- **Text** → `.primary` / `.secondary` (auto-invert). Replace every `.white.opacity(x)` with `.primary.opacity(x)` / `.secondary`.
- **Glass card — the official way (iOS 26):** use the real Liquid Glass API and **tint it dark** rather than faking glass with material — `.glassEffect(.regular.tint((isDark ? .black : .white).opacity(0.28)), in: shape)` keeps genuine specular + refraction while reading as a dark (or light) notification surface. Gate with `if #available(iOS 26.0, *)`; **fall back** to `.ultraThinMaterial` + a scheme-aware tint overlay (`isDark ? .black.opacity(0.45) : .white.opacity(0.35)`) on iOS 18–25. (A *plain untinted* `glassEffect` reads light — always tint for a dark surface; don't reach for material on iOS 26.)
- **Gradients / strokes / shadows** → branch on `isDark` (e.g. navy bg in dark, blue-white in light; shadow `0.35` dark vs `0.12` light).
- **Accent gradients** → lighten in dark, deepen in light, so the accent stays visible on both backgrounds.
- A stored `let` can't read the environment — make scheme-dependent values **computed `var`s**.

---

## iOS 26 Liquid Glass

Use `.glassEffect()` on iOS 26+, fall back to `.ultraThinMaterial` on older OS. Always wrap in a `@ViewBuilder` helper so both paths share the same call site:

```swift
@ViewBuilder
private func glassShape(radius: CGFloat) -> some View {
    if #available(iOS 26.0, *) {
        RoundedRectangle(cornerRadius: radius, style: .continuous)
            .fill(.clear)
            .glassEffect(in: RoundedRectangle(cornerRadius: radius, style: .continuous))
    } else {
        RoundedRectangle(cornerRadius: radius, style: .continuous)
            .fill(.ultraThinMaterial)
            .overlay(
                RoundedRectangle(cornerRadius: radius, style: .continuous)
                    .strokeBorder(Color.white.opacity(0.12), lineWidth: 1)
            )
    }
}
```

Apply to any shape — pill, capsule, circle. Never hardcode `.ultraThinMaterial` for new components when iOS 26 is a target.

### Tinted glass (colored states)

For destructive / accent buttons, use the built-in tint API — never layer a colored shape on top of glass:

```swift
.glassEffect(.regular.tint(dangerColor), in: Circle())
```

Danger red: `Color(red: 0.87, green: 0.32, blue: 0.42)` · Accent blue: `Color(red: 0.45, green: 0.65, blue: 1.0)`.

### `GlassEffectContainer` — morphing clusters

When multiple glass elements should fuse and separate with the iOS 26 metaball effect (e.g. a capsule that morphs into a separate circle), wrap them in a `GlassEffectContainer`:

```swift
@Namespace private var glassNS

GlassEffectContainer(spacing: 18) {
    HStack(spacing: 28) {                 // see spacing trap below
        capsule
            .glassEffect(in: Capsule())
            .glassEffectID("capsule", in: glassNS)

        if isExpanded {
            circle
                .glassEffect(.regular.tint(.red), in: Circle())
                .glassEffectID("circle", in: glassNS)
        }
    }
}
```

**Fusion-threshold spacing trap** — `GlassEffectContainer(spacing:)` is the fusion threshold, not visual padding. Any two glass elements within that distance will visually blob together permanently, even at rest. Always make the layout spacing **greater than** `containerSpacing`:

| containerSpacing | HStack spacing | Result |
|---|---|---|
| 18 | 10 | ❌ Permanent fusion — edges distorted at rest |
| 18 | 28 | ✅ Clean shapes at rest, metaball morph only during transition |

Rule: `layoutSpacing > containerSpacing + 6pt` for safe margin.

### Edge distortion trap

Never apply `.scaleEffect(x:y:anchor:)` with offset anchors (`.leading` / `.trailing`) to **individual** glass elements — subpixel rendering at the anchor edge causes visible edge distortion even when scale = 1.0. Apply container-wide rubber stretch instead:

```swift
GlassEffectContainer(spacing: 18) { ... }
    .scaleEffect(x: 1.0 + stretch * 0.05, y: 1.0 - stretch * 0.025, anchor: .center)
```

Also: don't add redundant `.clipShape(Capsule())` on top of `.glassEffect(in: Capsule())` — `glassEffect` already clips.

### 3-phase rubber-band toggle

For glass elements that pop apart or fuse on tap, stack 3 animations to feel like real rubber:

```swift
private func toggle() {
    HapticFeedback.mediumImpact()

    // Phase 1 — pre-stretch (rubber tensions)
    withAnimation(.spring(response: 0.28, dampingFraction: 0.55)) {
        stretchAmount = 1.0
    }
    // Phase 2 — morph fires at peak tension
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.05) {
        withAnimation(.spring(response: 0.55, dampingFraction: 0.72)) {
            isExpanded.toggle()
        }
    }
    // Phase 3 — release snap-back with overshoot
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.28) {
        withAnimation(.spring(response: 0.45, dampingFraction: 0.55)) {
            stretchAmount = 0
        }
    }
}
```

### Light backgrounds for glass demos

To showcase iOS 26 glass effects, use a light gradient background — dark backgrounds hide the refraction. Standalone demo:

```swift
LinearGradient(colors: [Color(white: 0.97), Color(white: 0.88)],
               startPoint: .topLeading, endPoint: .bottomTrailing)
    .ignoresSafeArea()
```

---

## SF Symbols Animation (iOS 17–26)

> **Auto-effect selection:** See Intent Inference § Symbol → Effect lookup above. The table there maps symbol names to the correct effect automatically — no need to specify `.drawOn`, `.breathe`, etc. in the prompt unless you want to override the default.

### `.drawOn` — iOS 26 only (SF Symbols 7)

**`isActive` is inverted.** This is the single most common mistake:

| isActive | Visual state | What plays |
|---|---|---|
| `true` | Hidden (pre-draw) | Nothing on initial render |
| `false` | Visible | drawOn traces stroke on |
| `false → true` | Hidden | Reverse (draw-off) plays automatically |

**Correct pattern — always name the state variable to reflect "hidden":**

```swift
@State private var isDrawSymbolHidden = true   // starts hidden

Image(systemName: "checkmark")
    .symbolEffect(.drawOn, options: .speed(0.5), isActive: isDrawSymbolHidden)
    .task {
        try? await Task.sleep(for: .seconds(0.5))
        while !Task.isCancelled {
            isDrawSymbolHidden.toggle()    // false → draws on; true → draws off
            try? await Task.sleep(for: .seconds(2.0))
        }
    }
```

**Symbol choice matters for `.drawOn`:**
- `heart.fill`, `star.fill` — flood-fill shapes, no stroke path. drawOn appears to jump from hidden to filled instantly. Not suitable for demonstrating the pen-trace effect.
- `checkmark`, `heart` (outline), `signature`, `wifi`, `star` (outline) — stroke-based, clearly show the trace.

Do NOT stack `.drawOn` and `.drawOff` on the same image. They conflict and keep the symbol invisible.

**`.drawOn` only plays on a *transition* — never on appear.** `isActive:` traces the path when the bound value *changes* while the view is visible. It does NOT replay when the view appears already in the visible state. Two consequences:

- **"Draw a … demo" / "animate …" → auto-play by default.** When the prompt asks the symbol to draw/animate itself (a showcase, a hint, a loading state), drive it with the `.task` loop above so it traces on its own. Only gate it behind a tap/state change when the prompt explicitly describes a user action ("tap to sign", "on submit"). Reaching for the interaction trigger when the user wanted a self-playing demo is the most common way a drawOn build looks "broken" — the API is present and correct, but nothing ever fires it.
- **If a tap drives it, the pad/area is empty until the first tap.** That's correct behavior, not a bug — but confirm it's what the prompt wanted.

**Verification trap — never validate a transition by pre-setting state to its destination.** Setting `@State private var isDrawSymbolHidden = false` at init renders the symbol *already fully drawn, with no transition* — so a screenshot shows a complete symbol and falsely confirms "it works." That validates layout, not motion. To actually verify the trace: launch with the auto-loop running (or script the tap), and capture a **mid-trace frame** (a partial stroke). A fully-drawn symbol in a screenshot proves nothing about whether the draw animated. This applies to every `isActive:`/`value:` symbolEffect, not just drawOn.

---

### Other Symbol Effects (iOS 17+)

```swift
// Discrete — fires once per value change (user actions, additions)
.symbolEffect(.bounce, value: count)
.symbolEffect(.wiggle, value: errorTrigger)

// Indefinite — runs while active (states: recording, loading, scanning)
.symbolEffect(.breathe, isActive: isRecording)
.symbolEffect(.variableColor.iterative.reversing, isActive: isLoading)
.symbolEffect(.rotate, isActive: isSyncing)

// Content transition — swapping symbols on state toggle (always needs withAnimation)
Image(systemName: isPlaying ? "pause.fill" : "play.fill")
    .contentTransition(.symbolEffect(.replace))
// trigger with: withAnimation(.smooth) { isPlaying.toggle() }
```

**Loop driver — use `.task`, not `Timer`:**
```swift
.task {
    while !Task.isCancelled {
        try? await Task.sleep(for: .seconds(interval))
        count += 1   // or toggle state
    }
}
```
`.task` cancels automatically when the view disappears. Timer leaks.

---

## Spring Presets

| Feel | Value |
|---|---|
| Snap / bounce | `.spring(response: 0.35, dampingFraction: 0.5)` |
| UI pop | `.spring(response: 0.35, dampingFraction: 0.6)` |
| Standard settle | `.spring(response: 0.4, dampingFraction: 0.65)` |
| Physics settle | `.spring(response: 0.45, dampingFraction: 0.7)` |
| Slow morph | `.spring(response: 0.6, dampingFraction: 0.8)` |
| Precision stiff | `.interpolatingSpring(stiffness: 220, damping: 22)` |
| Dial / scrub | `.interactiveSpring(response: 0.3, dampingFraction: 0.7)` |

Never use bare `.spring()` — always explicit `response` + `dampingFraction`. If user supplies values, use them verbatim. Map feel words: "stretchy" → 0.4/0.5, "snappy" → 0.35/0.6, "melts" → 0.6/0.8.

**Archetype → physics default** (use when the prompt doesn't name a feel):

| Archetype | Default physics |
|---|---|
| Symbol Showcase | `.easeInOut(duration: 0.4)` — symbol effects own their timing |
| Liquid Toggle | UI pop `0.35/0.6` |
| Gesture Card | Physics settle `0.45/0.7` |
| Sheet Interaction | Standard settle `0.4/0.65` |
| Data Dashboard | Dial/scrub `.interactiveSpring(response: 0.3, dampingFraction: 0.7)` |
| Glass Morph | 3-phase rubber-band (see iOS 26 section) |
| Particle / Physics Sim | No SwiftUI spring — custom physics loop |
| Loading Indicator | `.linear(duration: 0.8)` — never spring (loops look jittery) |

---

## Press Feedback — `PressableScale`

Every tappable row/button should shrink under the finger and feel like a real press. Use a **gesture-driven** modifier, not `Button`/`ButtonStyle` — `ButtonStyle.isPressed` cannot fire a haptic on the press-*down* edge, but a `DragGesture(minimumDistance: 0)` can.

```swift
private struct PressableScale: ViewModifier {
    var pressedScale: CGFloat = 0.96
    let action: () -> Void
    @State private var isPressed = false

    func body(content: Content) -> some View {
        content
            .scaleEffect(isPressed ? pressedScale : 1.0)
            .animation(.spring(response: 0.3, dampingFraction: 0.6), value: isPressed)
            .contentShape(Rectangle())
            .simultaneousGesture(                       // simultaneous → won't block parent scroll
                DragGesture(minimumDistance: 0)
                    .onChanged { _ in
                        if !isPressed {                 // haptic on the DOWN edge only
                            isPressed = true
                            HapticFeedback.lightImpact()
                        }
                    }
                    .onEnded { v in
                        isPressed = false
                        // lift-inside-to-fire: dragging off cancels, like a real UIButton
                        if abs(v.translation.width) < 20, abs(v.translation.height) < 20 { action() }
                    }
            )
    }
}
extension View {
    func pressable(scale: CGFloat = 0.96, action: @escaping () -> Void = {}) -> some View {
        modifier(PressableScale(pressedScale: scale, action: action))
    }
}
```

Non-obvious rules baked in:
- **`.simultaneousGesture`** (not `.gesture`) so a row inside a `ScrollView`/`List` still scrolls.
- **Haptic on the down edge only** (`if !isPressed`) — not on every `onChanged` tick.
- **Lift-inside-to-fire** — verify the finger lifted within ~20pt before running `action`; a drag-away must cancel.
- `pressedScale` ≈ `0.90` for round icon buttons, ≈ `0.965` for wide rows.

---

## Entrance / Appear Animation

Premium screens *assemble themselves*. Stagger children in on appear with a single `@State` flag.

```swift
@State private var appeared = false

ForEach(Array(items.enumerated()), id: \.element.id) { i, item in
    rowView(item)
        .opacity(appeared ? 1 : 0)
        .offset(y: appeared ? 0 : 16)
        .scaleEffect(appeared ? 1 : 0.97, anchor: .top)
        .animation(.spring(response: 0.5, dampingFraction: 0.82)
                       .delay(0.10 + Double(i) * 0.07), value: appeared)
}
.onAppear { appeared = true }   // trigger once — NEVER init appeared = true
```

- Combine `opacity + offset(y:) + scaleEffect(anchor:.top)` for a soft rise-and-settle; `0.07s` per-index delay reads as a cascade.
- **Inside a `.sheet`, `@State` resets on every presentation** — so this re-fires each time the sheet opens, a free "assembles itself" entrance with no extra code.
- Verification trap (same as `.drawOn`): never initialise the flag to its destination — the transition won't play.

---

## Custom Transitions — rubber-band entrance

`.transition(.scale)` only interpolates from its scale **to 1.0** — so it can't make an element enter *oversized* and settle back. For an "enters wider, rubber-bands to true size" pop, write a custom modifier transition and drive it with a **low-damping (bouncy) spring** so it overshoots:

```swift
private struct WidthPopModifier: ViewModifier {
    let active: Bool
    func body(content: Content) -> some View {
        content
            .scaleEffect(x: active ? 1.16 : 1.0, y: active ? 1.07 : 1.0, anchor: .top)  // anisotropic → wider
            .offset(y: active ? -34 : 0)
            .opacity(active ? 0 : 1)
    }
}
private extension AnyTransition {
    static var widthPop: AnyTransition {
        .modifier(active: WidthPopModifier(active: true), identity: WidthPopModifier(active: false))
    }
}

// usage — the surrounding withAnimation's spring governs the transition:
withAnimation(.spring(response: 0.5, dampingFraction: 0.55)) { items.insert(item, at: 0) }
// on the view:  .transition(.asymmetric(insertion: .widthPop, removal: .move(edge: .top).combined(with: .opacity)))
```

- **Anisotropic scale** (`x > y`) makes it read as a *width* stretch, not a uniform zoom.
- The rubber-band comes from the **spring damping**, not the transition: `dampingFraction ≈ 0.55` overshoots; `≈ 0.8` settles flat. Lower = bouncier.
- `.modifier(active:identity:)` is the general recipe for any entrance the built-in transitions can't express.

---

## Data Dashboard Patterns

- **Animated proportional bar** — capsule track + fill whose width grows from 0 on appear and re-springs on data change:
```swift
GeometryReader { geo in
    ZStack(alignment: .leading) {
        Capsule().fill(track).frame(height: 6)
        Capsule().fill(ink.opacity(0.85))
            .frame(width: geo.size.width * (appeared ? share : 0), height: 6)
            .animation(.spring(response: 0.6, dampingFraction: 0.85), value: appeared)
            .animation(.spring(response: 0.5, dampingFraction: 0.8), value: selection)
    }
}.frame(height: 6)
```
- **`.numericText` as a count-up** — not just for currency. A hero metric that changes with a segmented selector rolls its digits via `.contentTransition(.numericText(value: Double(total)))` + `.animation(..., value: period)`. Pair the segment switch with `selectionChanged` haptic and a sliding indicator (see Tab Bar Patterns).

---

## State & Code Rules

- Default: pure `@State` for all gesture tracking · `@GestureState` only if value must auto-reset
- Constants: camelCase `private let` at struct level (2–5 values) — never SCREAMING_SNAKE_CASE
- Liquid metaball: render blobs in a `Canvas` with `.addFilter(.alphaThreshold)` + `.addFilter(.blur)` — see the **Canvas Metaball** section. (Older alt: white circles on black + `.blur` + `.contrast(50)` + `.blendMode(.screen)`.)
- Multi-phase sequences: stacked `DispatchQueue.main.asyncAfter` with overlapping delays

**ZStack frame trap — always apply both rules together:**
When a ZStack has a fixed `.frame(height:)` AND contains a `LazyVGrid`, `List`, or any tall component:
1. Conditionally render the tall child only when needed (`if isExpanded || childVisible`)
2. Add `.clipped()` to the ZStack

Relying on `.opacity(0)` alone does NOT prevent overflow — the component still renders outside the frame and changing `height` constants has no visual effect.

**Floating bar layout:** Build multi-component bars (pill + separate action button) as `HStack` of independent views, not one monolithic ZStack. Each component gets its own glass background and shadow.

**Sheet wrapping — outer launcher + private inner content:**
When a demo needs a native `.sheet`, use two structs in the same file. `private` model types defined in the file are visible to both structs — no access-level changes needed.

```swift
struct MyDemoView: View {               // public — registered in ContentView
    @State private var showSheet = false
    var body: some View {
        ZStack { Color("BgColor").ignoresSafeArea(); triggerButton }
            .sheet(isPresented: $showSheet) {
                MySheetContent()
                    .presentationDetents([.height(540)])
                    .presentationCornerRadius(28)
                    .presentationDragIndicator(.visible)
            }
    }
}

private struct MySheetContent: View {  // private — only used as sheet body
    @Environment(\.dismiss) private var dismiss
    // sheet body here — X button calls dismiss()
}
```

**Numeric text transition — split the currency prefix:**
`.contentTransition(.numericText(value:))` on a currency string like `"$173.11"` can glitch the `$` during the digit roll. Split prefix and number into separate `Text` views — only the numeric part gets the transition:

```swift
HStack(alignment: .firstTextBaseline, spacing: 1) {
    Text("$")                                          // static — no transition
        .font(.system(size: 32, weight: .bold))
    Text(String(format: "%.2f", amount))
        .font(.system(size: 44, weight: .bold))
        .contentTransition(.numericText(value: amount))
        .animation(.spring(response: 0.4, dampingFraction: 0.65), value: triggerValue)
}
```

When the transition is driven by a discrete step (card carousel, dial tick): use `selectionChanged` haptic, not `mediumImpact` — it matches the numeric ticker feel.

**File layout (mandatory MARK order):**
```
// MARK: - Model
// MARK: - Main View   (tokens → @State → body → subviews → gesture → actions)
// MARK: - Supporting Shapes
// MARK: - Preview
```

---

## Archetype Catalog

The archetype drives physics, haptics, and container defaults. Pick the closest match from the prompt — if ambiguous, choose the simpler one.

| Archetype | Prompt signals | Container | Physics | Haptics |
|---|---|---|---|---|
| **Symbol Showcase** | "animate", "draw", "show" + symbol name | none (inline) | `.easeInOut` | none |
| **Liquid Toggle** | "toggle", "switch", "flood", "metaball" | inline | UI pop `0.35/0.6` | mediumImpact on toggle |
| **Gesture Card** | "drag", "pull", "rip", "swipe", "dismiss" | inline ZStack | Physics settle `0.45/0.7` | light start → heavy commit |
| **Sheet Interaction** | "sheet", "sign", "draw", "pick", "form" | `.sheet` | Standard settle `0.4/0.65` | selectionChanged on controls |
| **Data Dashboard** | "chart", "graph", "metric", "scrub", "data" | full-screen | Dial/scrub `.interactiveSpring` | selectionChanged per step |
| **Glass Morph** | "glass", "fuse", "morph", "capsule" | inline | 3-phase rubber-band | mediumImpact on morph |
| **Particle / Physics Sim** | "gravity", "fluid", "rope", "cloth", "sand" | full-screen | custom physics loop | lightImpact on touch |
| **Loading Indicator** | "loading", "spinner", "progress", "scanning" | inline | `.linear(duration:)` | none |

Print the resolved archetype on the `🎯  Archetype:` line. If the user's prompt overrides any default in this table, use their value and note the override in parentheses.

---

## Tab Bar Patterns

**Sliding selection indicator** — use `GeometryReader` + `offset(x:)`, never manual position math:

```swift
GeometryReader { geo in
    let tabW = geo.size.width / CGFloat(tabCount)
    RoundedRectangle(cornerRadius: indicatorRadius, style: .continuous)
        .fill(Color.white.opacity(0.14))
        .padding(5)
        .frame(width: tabW)
        .offset(x: tabW * CGFloat(selectedIndex))
        .animation(.spring(response: 0.35, dampingFraction: 0.65), value: selectedIndex)
}
```

**Active indicator spec** (matches Apple Music / iOS 26 HIG):
- Fill: `Color.white.opacity(0.14)` — solid presence, clearly distinct from glass background
- Padding: `5pt` inset on all sides (fills full tab height minus 5pt each edge)
- Corner radius: `outerRadius - 5` (softer than the outer pill, independent value ~16–18pt)
- Width: exactly `pillWidth / tabCount` — crisp slot division, no guessing

**Tab cell layout** (icon + label, per HIG):
- Icon: 22pt, `.semibold` when active · `.regular` inactive
- Label: 10–11pt, `.semibold` when active · `.regular` inactive
- Both tinted with `activeColor` when selected · `white.opacity(0.5)` inactive
- `VStack(spacing: 4)` inside `.frame(maxWidth: .infinity).frame(height: barHeight)`

**SF Symbol replace transition** — for any button that toggles state and changes its icon (e.g. more → close, play → pause, add → done):

```swift
Image(systemName: isExpanded ? "xmark" : "ellipsis")
    .contentTransition(.symbolEffect(.replace.downUp))
    .animation(.spring(response: 0.35, dampingFraction: 0.6), value: isExpanded)
```

- `.replace.downUp` — old icon scales down, new scales up. Use for open/close toggles.
- `.replace.upUp` — both scale up. Use for sequential forward actions.
- The button action must toggle: `isExpanded ? collapse() : expand()` — never hide the button to show a separate close button.
- Available iOS 17+. No fallback needed for legendary-Animo (iOS 16+ minimum → gate with `if #available(iOS 17, *)` only if targeting iOS 16).

**Standard heights:** `barHeight = 49pt` (icon-only) · `barHeight = 68pt` (icon + label)

---

## Carousels & Paging

**Velocity-aware paging — threshold on `predictedEndTranslation`, not raw translation.** A slow short drag shouldn't page; a fast flick should — even if the finger barely moved. The predicted end is velocity-aware:

```swift
DragGesture(minimumDistance: 12)
    .onChanged { v in dragOffset = v.translation.width * 0.5 }   // 0.5 = drag resistance
    .onEnded { v in
        let threshold: CGFloat = 52
        withAnimation(.spring(response: 0.42, dampingFraction: 0.72)) {
            if v.predictedEndTranslation.width < -threshold, index < count - 1 {
                index += 1; HapticFeedback.selectionChanged()
            } else if v.predictedEndTranslation.width > threshold, index > 0 {
                index -= 1; HapticFeedback.selectionChanged()
            }
            dragOffset = 0
        }
    }
```

**Fan / card-stack layout** — position each card from its distance to the selected index, scale down siblings, z-order by proximity:

```swift
let diff = CGFloat(index - selectedIndex)
card
    .scaleEffect(index == selectedIndex ? 1.0 : 0.86)
    .offset(x: diff * xStep + dragOffset, y: index == selectedIndex ? -10 : 14)
    .zIndex(index == selectedIndex ? 10 : Double(5 - abs(index - selectedIndex)))
    .animation(.spring(response: 0.42, dampingFraction: 0.72), value: selectedIndex)
```

- `selectionChanged` (not `mediumImpact`) per card switch — discrete scrub feel.
- Apply a **resistance multiplier** (`* 0.5`) to `dragOffset` so the stack feels weighty under the finger.
- `MeshGradient` (iOS 18+, 9-point) makes a premium card/orb fill — animate the control points for a living surface.

---

## Stacked Cards (notification stack)

Apple's collapsed notification stack: newest card full-size in front, older ones **peek behind** (scaled down + offset + dimmed by depth), tap to **expand** into a list, **swipe the front card to dismiss**, and **cap** the count so the oldest drops off the back.

```swift
// front = index 0. Transform each card from its depth + an `expanded` flag:
let yOffset  = expanded ? CGFloat(depth) * (cardHeight + 10) : CGFloat(depth) * 16   // peek step
let scale    = expanded ? 1.0 : max(0.88, 1.0 - CGFloat(depth) * 0.06)
let opacity  = expanded ? 1.0 : (depth < 3 ? 1.0 - Double(depth) * 0.22 : 0.0)        // only ~3 peek
card.scaleEffect(scale).opacity(opacity).offset(y: yOffset).zIndex(Double(count - depth))
```

- **Insert front + cap:** `withAnimation(.spring) { items.insert(new, at: 0); if items.count > cap { items.removeLast() } }` — newest pops in (pair with `widthPop`), oldest fades off the back.
- **Expand toggle:** `onTapGesture` on the stack flips `expanded` with `selectionChanged` haptic + `.spring(0.5/0.8)`.
- **Swipe-to-dismiss (front only):** gate the `DragGesture` to `depth == 0`; rubber-band `dragOffset`, `lightImpact` on start, past ~120pt `removeFirst()` + `heavyImpact`, else spring back.
- Each card `.transition(.asymmetric(insertion: .widthPop, removal: .move(edge: .top).combined(with: .opacity)))`.
- **"Notification" card surface:** on iOS 26 use authentic Liquid Glass — `.glassEffect(.regular.tint((isDark ? .black : .white).opacity(0.28)), in: shape)`; fall back to `.ultraThinMaterial` + scheme-aware tint below (see Light Theme → Adaptive). 28pt continuous radius + soft shadow.

---

## Canvas Metaball (goo / liquid extrusion)

For elements that should **fuse and split like liquid in plain SwiftUI** (no iOS 26 `GlassEffectContainer`) — e.g. a speed-dial FAB whose actions extrude out of the button — render the blobs in a `Canvas` with the goo filter:

```swift
Canvas { ctx, size in
    ctx.addFilter(.alphaThreshold(min: 0.5, color: blobColor)) // hard edge after blur
    ctx.addFilter(.blur(radius: 16))                           // halos overlap → merge
    ctx.drawLayer { layer in
        layer.fill(Path(ellipseIn: fabRect), with: .color(.white))
        for b in bubbles { layer.fill(Path(ellipseIn: b.rect), with: .color(.white)) }
    }
}
```

Non-obvious rules — each one is a real trap:

- **`alphaThreshold` gives a HARD edge.** Blobs are crisp circles at rest, gooey only where they overlap. "Liquid" ≠ "soft/blurry".
- **Animate the Canvas by making the layer `View, Animatable`** with `animatableData = progress`. A raw `@State` read inside the Canvas closure *jumps* — Animatable lets SwiftUI interpolate `progress` frame-by-frame and redraw the goo each step.
- **Keep every blob ~the same size.** One global blur only matches one circle size — if a bubble shrinks (e.g. `0.45·r` mid-travel) the fixed blur ≈ its radius and the threshold collapses it into a **line**. Hold blobs near full size (`0.9–1.0·r`).
- **Rest spacing must exceed `diameter + 2·blur`** or neighbours never clear each other's blurred alpha → permanent **teardrops**. The neck should exist only *during* the transition.
- **Slow the morph** (`spring(response ≈ 1.0)`) — fast springs hide the whole goo.
- **The Canvas is visual only.** Put real `Button`s on top at the same positions for taps + icons. Icons must **travel with their blob on the same spring** (animated `.position`), not `.transition`-pop at the destination, or the motion looks non-uniform. Closed buttons stacked on the FAB need `.allowsHitTesting(false)` so they don't steal its tap.

**Styling taste:** a liquid FAB reads best **flat** (no shadow). Make blob + icon colors parameters that invert with the background — dark bg → white blob + dark glyphs, light bg → dark blob + white glyphs. `+` → `✕` is just a `135°` rotation of one `plus` symbol; a `.thin` weight makes a premium glyph.

---

## Canvas Path Loaders (outline tracing / comet trail)

For a loader that **traces a shape's outline with a fading comet trail** (infinity, star, polygon, heart, rose…), drive a `Canvas` from a `TimelineView` clock and a *precomputed* set of outline samples — never re-evaluate the shape's math every frame:

```swift
// Build ONCE — cache unit samples per shape (e.g. in a static dictionary).
let samples: [CGPoint] = makeSamples(for: shape)   // arc-length-even points

TimelineView(.animation(minimumInterval: 1.0/60.0)) { ctx in
    let progress = (ctx.date.timeIntervalSince(start) / duration).wrappedUnit
    Canvas(rendersAsynchronously: true) { gc, size in
        // stroke the static outline faintly …
        // … then draw N trail dots, each at interpolatedPoint(in: samples,
        //    progress: progress - tail*trailLength), radius/opacity fading by tail.
    }
}
```

Non-obvious rules — each one is a real trap:

- **Sample-then-animate; never eval-per-frame.** Precompute the outline as `[CGPoint]` unit samples (cache them). Animate only a `progress` head that **interpolates along the cached samples**. The renderer (outline stroke + trail) is shape-agnostic — adding a new shape = adding one sample generator, with *zero* changes to the motion code. Keeps "what shape" fully decoupled from "how it moves."
- **Arc-length-even sampling = constant-velocity motion.** Walk the outline at equal *arc-length* steps, not equal parameter `t`. Equal-`t` bunches points where the curve is slow and spreads them where it's fast, so a head advancing by `progress` visibly **speeds up / slows down** (worst around polygon corners). Equal arc-length → the head glides at uniform speed. **Shape sharpness and motion smoothness are independent** — sharp corners in the silhouette do *not* cause jerk in the travel.
- **Two samplers for two silhouettes.** A polyline arc-length sampler (straight edges, crisp vertices → star / polygon) vs a closed **Catmull-Rom** spline (organic flowing curves). Catmull-Rom needs `> 3` control points — fall back to the polyline sampler otherwise. Pick by whether the shape *should* have corners.
- **Shape recipes.** An N-point **star** = `2N` vertices alternating outer radius `1.0` / inner radius `~0.42`; a **regular polygon** = `N` unit-circle vertices; a rotation offset orients it (point-up `-π/2`, flat-top hexagon `+π/6`).
- **Use a `TimelineView` clock, not a spring / `withAnimation`.** A loader is a continuous loop, not a state transition — derive everything from `time`: `progress = (time/duration).wrappedUnit`, plus independent `rotation` and `breathe` from the same clock. (This is the Canvas form of the **Loading Indicator → never spring** rule in the Archetype Catalog.)

**Content taste:** for a generic loader showcase, prefer **generic geometric primitives** (star, pentagon, hexagon, infinity) over tracing brand/trademark logos — same premium feel, no trademark exposure, and the shapes generalize.

---

## Output (Create mode)

Stream these progress lines one by one:

```
⚙️  swiftui-microinteractions v1.11.0
🖼️  Assets: <found: name1, name2… · or · none found, using placeholders>
🎯  Archetype: <archetype name>
⚡  Physics: <spring preset and why — one phrase>
🎮  Haptics: <2–3 haptic moments>
🏗️  State: <tier> — <@State var names>
✍️  Writing <FileName>.swift…
```

**Step 1 — Write the file to disk:**
- Carousel → `legendary-Animo/Carousels/` if exists, else `Carousels/` if exists, else cwd
- Other → `legendary-Animo/Animations/` if exists, else `Animations/` if exists, else cwd

Print: `✅  Saved to <full relative path>`

---

**Step 2 — Xcode project registration:**

Check for a `.xcodeproj` file:
```bash
find . -maxdepth 3 -name "*.xcodeproj" -type d | head -1
```

**If `.xcodeproj` found** → register the file in `project.pbxproj` using this Python script:

```python
import uuid, re

pbxproj  = "<xcodeproj_path>/project.pbxproj"
filename = "<FileName>.swift"
filepath = "<full relative path written above>"
# Determine target group: "Carousels" if saved in Carousels/, else "Animations"
group_name = "Carousels"  # or "Animations"

FILE_REF   = uuid.uuid4().hex[:24].upper()
BUILD_FILE = uuid.uuid4().hex[:24].upper()

with open(pbxproj) as f:
    content = f.read()

# Find an existing file in the same group as an insertion anchor
# Look for the last .swift entry in PBXFileReference section for that group
# Insert PBXBuildFile, PBXFileReference, group child, and Sources build phase entry
# Use the same indentation and formatting as surrounding entries
```

Run the script with the actual values. Use an existing same-group file as the anchor for each insertion point. Print:

```
📦  Registered in Xcode project: <xcodeproj name>
    └─ PBXFileReference  ✓
    └─ PBXBuildFile      ✓
    └─ Group child       ✓
    └─ Sources phase     ✓
```

**If no `.xcodeproj` found** → print:

```
⚠️  No Xcode project found. Add the file manually:
    File path: <full path>
    In Xcode: File → Add Files to "<ProjectName>"
              or drag into the Navigator → check "Add to target"
```

---

**Step 3 — ContentView registration snippet** (```swift block, paste manually):

Use today's actual date for the `date:` field — never hardcode a past date.

```swift
DemoItem(
    row: RowView(icon: "💧", title: "Title Here", desc: "Feature · feature · feature"),
    destination: wrappedDestination { YourView() },
    date: "<today's date e.g. June 11, 2026>",
    hasDateHeader: true
)
```

**Step 4 — Asset notes** (only if placeholders were used):
```
ASSET NOTES:
- Add portrait images to Assets.xcassets/Images/ named: photo1, photo2…
  → Recommended size: 400×600pt, portrait ratio
- Remove placeholder RoundedRectangle once real assets are added
```

---

## Output (Edit mode)

Stream before editing:
```
📂  Reading <FileName>.swift…
✏️  Applying: <one-line summary of change>
✍️  Writing updated file…
```

Overwrite the file at its existing path, then print:
```
✅  Updated <full relative path>
```

Then output a **Changes** bullet list of what was modified and why.
(No pbxproj update needed for edits — file is already registered.)
