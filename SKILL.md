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
| liquid chrome / molten metal / mercury / brushed steel | Metal Shader — stitchable `.colorEffect` (see Metal Shaders section) |
| holographic / iridescent / oil-slick / foil / prism | Metal Shader — fresnel rainbow in a `.colorEffect` |
| plasma / lava / nebula / aurora / fluid ink | Metal Shader — FBM field in a `.colorEffect` |

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

**Gate-and-rearm threshold haptics — fire once per crossing, not once ever.** A drag that crosses a dismiss/commit threshold, retreats, then crosses again should buzz *every* crossing, not just the first. A plain "fired" boolean that's never reset only fires once for the life of the gesture. Gate on entry, **rearm on exit**:

```swift
@State private var didCrossThreshold = false
// in onChanged:
if !didCrossThreshold, y > threshold {
    didCrossThreshold = true
    impact(.medium)                     // fires on every fresh crossing
} else if didCrossThreshold, y <= threshold {
    didCrossThreshold = false           // rearm — retreating past the line resets it
}
// reset both didCrossThreshold and any didStartDrag flag in onEnded
```

Apply the same gate (without rearm, since it only happens once) to the touch-down `lightImpact`, with a small dead-zone (`y > 2`) so sub-pixel jitter at gesture start doesn't fire it prematurely.

**SourceKit `HapticFeedback` false positive — always ignore:**
In files written to `Carousels/` or `Animations/`, SourceKit reports `Cannot find 'HapticFeedback' in scope`. This is **not a real error** — SourceKit analyzes the new file in isolation and doesn't see other module members. `HapticFeedback.swift` is registered in the Sources build phase; the real compiler resolves it correctly. Do not add `import UIKit`, do not re-declare the struct, do not alter the code.

---

## Style Rules

**Backgrounds:** `Color(white: 0.06).ignoresSafeArea()` standalone · `Color("BgColor").ignoresSafeArea()` in-project

**Glass surfaces (pre-iOS 26):** `.ultraThinMaterial` or `.thickMaterial` clipped to shape · track background `.white.opacity(0.06)` + 1pt stroke `.white.opacity(0.08)`

**Opacity levels:** ghost `0.06–0.08` · subtle `0.12` · inactive `0.3` · secondary `0.5` · active `0.8–0.9` · full `1.0`

**Colors:** white + opacity dominant · cyan `Color(red: 0.45, green: 0.65, blue: 1.0)` · green `Color(red: 0.55, green: 0.95, blue: 0.75)` · gold/rating `Color(red: 1.0, green: 0.84, blue: 0.35)` · never `.blue/.green/.red` on dark bg

**Gloss overlay (photographic cards):** a diagonal sheen on a poster/photo card reads as premium glass without touching the image — overlay a `LinearGradient(colors: [.white.opacity(0.18), .clear], startPoint: .topLeading, endPoint: .center)` clipped to the card shape with `.blendMode(.softLight)`. `.softLight` (not `.screen`) keeps the highlight subtle enough not to wash out the photo underneath.

**Image-colored ambient shadow (glow):** a plain black `.shadow` under a poster/photo card reads as generic weight, not light. For a shadow that feels like it belongs to that specific artwork, duplicate the image itself behind the card, heavily blurred and dimmed:

```swift
.background {
    Image(item.imageName)
        .resizable().scaledToFill()
        .frame(width: cardWidth, height: cardHeight)
        .clipShape(cardShape)
        .blur(radius: 34)
        .opacity(0.65)
        .offset(y: 22)
        .scaleEffect(0.96)   // slightly smaller so the blurred edge doesn't peek past the card
}
```
The result is a soft, color-tinted halo instead of a neutral drop shadow — reserve it for hero cards (front of a deck, selected item), not every row, since it's a duplicate render of the image. Works for any image-backed card — poster, album art, product photo.

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

**When the prompt says "Liquid Glass" → use the GENUINE effect, always.** Two non-negotiables:

- **Real API, never a look-alike.** Use real `.glassEffect(.regular)` (or `.regular.interactive()` for a pressable control) on iOS 26. Do **not** fake it with a near-opaque fill as the *primary* surface — a `Capsule().fill(.ultraThinMaterial)` or a heavy `.glassEffect(.regular.tint(.white.opacity(0.5)))` reads as a flat chip, not glass. `.ultraThinMaterial` is the **< iOS 26 fallback only**, gated behind `if #available(iOS 26.0, *)`.
- **Put busy, colorful content BEHIND the glass** so the refraction/specular actually reads — an image grid, a photo wall, a vivid gradient. On a flat solid or pale background Liquid Glass is nearly invisible and looks like plain material; that's the #1 reason a "glass" build looks wrong. (For a standalone showcase, an image-tile grid backdrop is the safest way to make the effect pop.)

**`.regular` vs `.clear`:** `.glassEffect(.regular, in:)` is the default frosted intensity — reach for `.glassEffect(.clear, in:)` on a bar/control that sits over especially rich, busy, high-contrast content (a poster, a photo backdrop) where `.regular`'s frosting would flatten the artwork more than wanted. `.clear` lets more of the backdrop's color and detail through.

**Never put a live `.glassEffect()` on a chip riding a view that's actively being dragged or animated fast** (a meta pill on a card mid-swipe, a badge on a scrubber thumb). Glass re-samples its backdrop continuously, and on a fast-moving element that resampling reads as the glass visibly **growing or pulsing** rather than looking stable — not a subtle bug, an obviously broken-looking one. Use a plain static translucent fill instead (`Color.black.opacity(0.45)` in the same shape) for anything attached to actively-moving content; reserve real glass for elements that are static or move slowly.

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
| **Metal Shader** | "liquid chrome", "molten metal", "holographic", "plasma", "shader" | full-screen | `TimelineView` clock (never spring) | lightImpact on touch |
| **Loading Indicator** | "loading", "spinner", "progress", "scanning" | inline | `.linear(duration:)` | none |
| **3D Object Showcase** | "3D", "SceneKit", "spin the cover", "album", "boxed product" | embedded or full-screen | SceneKit rig, `SCNTransaction.disableActions` for driven properties (never spring) | selectionChanged per settle / none if passive |

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

**Expanding tab bar (content-driven width, no indicator math)** — when the bar doesn't need fixed-width cells (e.g. a floating glass capsule over content, not a full-width screen bar), skip the `GeometryReader`/indicator entirely: only the selected tab shows its label, so its `Button` naturally grows and the container reflows around it.

```swift
HStack(spacing: 10) {
    ForEach(NavTab.allCases, id: \.self) { tab in
        let isSelected = tab == selectedTab
        Button { select(tab) } label: {
            HStack(spacing: 6) {
                Image(systemName: tab.symbol).symbolEffect(.bounce, value: isSelected)
                if isSelected { Text(tab.title) }   // label only exists when selected — no width reserved
            }
            .foregroundStyle(.white.opacity(isSelected ? 1 : 0.6))
            .padding(.horizontal, 14).padding(.vertical, 9)
        }
        .buttonStyle(.plain)
    }
}
.padding(6)
.glassEffect(.clear, in: .capsule)                                    // capsule autosizes to content
.animation(.spring(response: 0.4, dampingFraction: 0.8), value: selectedTab)
```

Simpler than the sliding-indicator recipe when the bar floats over rich content rather than filling the screen width — there's no slot math because there are no fixed slots. Trade-off: tabs shift position as labels appear/disappear, so it reads as "compact icon bar with one tab announcing itself," not a stable grid — pick the sliding-indicator version instead when tabs must stay in fixed positions.

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

### Coverflow / continuous depth fan

The two-state fan above (`selected` vs `not`) is enough for a simple stack, but a **"flip through posters/cards" coverflow** wants every card — not just the immediate neighbor — to recede continuously into the distance, and to do so *while the finger is still dragging*, not just after the gesture ends. Drive every transform off one **signed, drag-blended distance** instead of a boolean:

```swift
private func effectiveDistance(for i: Int) -> CGFloat {
    CGFloat(i - selectedIndex) - (dragOffset / cardWidth)   // blends live drag in
}

private func scale(for i: Int)    -> CGFloat  { max(0.82, 1.0 - 0.12 * abs(effectiveDistance(for: i))) }
private func rotation(for i: Int) -> Double   { Double(max(-2, min(2, effectiveDistance(for: i))) * 7) }  // cap at ±14°
private func yOffset(for i: Int)  -> CGFloat  { abs(effectiveDistance(for: i)) * 18 }
private func opacity(for i: Int)  -> Double   { max(0.0, 1.0 - 0.28 * abs(effectiveDistance(for: i))) }
private func blur(for i: Int)     -> CGFloat  { min(4, abs(effectiveDistance(for: i)) * 2) }

card
    .scaleEffect(scale(for: i))
    .rotationEffect(.degrees(rotation(for: i)))
    .offset(x: CGFloat(i - selectedIndex) * xStep + dragOffset, y: yOffset(for: i))
    .opacity(opacity(for: i))
    .blur(radius: blur(for: i))
    .zIndex(-abs(effectiveDistance(for: i)))   // closest to center draws on top
```

- **Divide `dragOffset` by card width, not a magic constant** — that normalizes the drag into "fractional cards moved," so neighbors visibly grow/shrink in real time as the finger moves, then settle the rest of the way once the spring takes over on release. Without this the fan only updates in a jump when the gesture ends.
- **Cap rotation and clamp scale/opacity to a floor** (`max(-2, min(2, d))`, `max(0.82, …)`) — beyond ~2 cards away the linear formulas would flip cards past vertical or invert opacity; clamping keeps distant cards small, dim, and blurred instead of glitching.
- **Blur-by-distance is a cheap depth-of-field** — 2–4pt of blur on off-center cards sells "depth" far more than scale/opacity alone, and is nearly free compared to real depth-of-field rendering.
- Use `selectionChanged` haptic on index change (see velocity-aware paging above); the continuous transform is purely visual and doesn't change the haptic ladder.
- **Asymmetric response is fine — and often more physical — when the two directions should feel different.** Nothing requires the distance-response curves to be symmetric: a stack of cards "already flipped past" vs. "still ahead" can use different degrees-per-step (e.g. ahead fans flatter, behind stands more upright), so the two sides of the deck read as physically distinct rather than a mirror image of each other. Branch the formula on the sign of the distance, not just its magnitude.

### Selection-synced immersive backdrop

For a "browse and the whole screen reacts" feel (movie posters, album art, product photos), cross-fade a **full-bleed blurred backdrop** behind the carousel that always matches the selected card:

```swift
GeometryReader { geo in
    ZStack {
        ForEach(items.indices, id: \.self) { i in
            Image(items[i].imageName)
                .resizable().scaledToFill()
                .frame(width: geo.size.width, height: geo.size.height)
                .clipped()
                .opacity(i == selectedIndex ? 1 : 0)
                .animation(.easeInOut(duration: 0.55), value: selectedIndex)
        }
    }
    .blur(radius: 80, opaque: true)   // opaque: true — avoids edge transparency artifacts at high radius
    .saturation(1.3)                  // blur flattens color; push saturation back up
}
```

- **Stack all candidate images and cross-fade opacity — don't swap the `Image` view.** Swapping which image is in the tree restarts its load/layout and can't cross-fade; toggling `opacity` on views that are always present animates smoothly and is what makes `.easeInOut(duration: 0.55)` read as a dissolve.
- **`.blur(radius:opaque:)` with `opaque: true`** is both faster and avoids the translucent-edge artifact that plain `.blur` produces at large radii on a full-bleed image.
- **Apply `.blur`/`.saturation` once to the whole `ZStack`**, not per-image — one blur pass instead of N.
- Overlay a top/bottom `LinearGradient` scrim (`black.opacity(0.55) → clear → black.opacity(0.70)`) so header and title text stay legible over any photo.
- **Two ways to fade it out — scrim vs. mask, pick by what's underneath.** The scrim above (a translucent gradient laid *on top*) is right when there's readable content over the *whole* backdrop. When the backdrop should only occupy the **top** portion of the screen and hand off cleanly to a solid-color card below it, `.mask` the image itself with a `white → clear` gradient instead: `.mask(LinearGradient(stops: [.init(color: .white, location: 0), .init(color: .white, location: 0.18), .init(color: .clear, location: 0.42)], startPoint: .top, endPoint: .bottom))` over a plain `Color.black` behind it. A mask *reveals the layer behind it* (true transparency down to black), where a scrim only *dims* the image — use mask when you need a hard, clean handoff to a solid surface; use scrim when text needs to sit legibly on top of photo for the full extent.

### Page indicator dots (variable-width)

For a lightweight position indicator under a carousel/paging view (simpler than the Tab Bar sliding indicator, since there's no tab content to align to):

```swift
HStack(spacing: 7) {
    ForEach(items.indices, id: \.self) { i in
        Capsule()
            .fill(.white.opacity(i == selectedIndex ? 0.95 : 0.3))
            .frame(width: i == selectedIndex ? 22 : 7, height: 7)
    }
}
.animation(.spring(response: 0.4, dampingFraction: 0.7), value: selectedIndex)
```

The active dot **stretching into a capsule** (22pt vs a 7pt circle) rather than just changing color/opacity is what makes it read as premium — a plain color-only dot indicator looks dated by comparison.

### Detail text swap on selection (lightweight alternative to `.numericText`)

When a carousel/list selection drives adjacent text (title, metadata) that isn't numeric, `.numericText` doesn't apply — use `.contentTransition(.opacity)` keyed to an `.id()` that changes with the selection instead of a custom transition:

```swift
Text(current.title)
    .contentTransition(.opacity)
    .id(current.id)                              // changing id forces the transition to fire
    .animation(.easeInOut, value: current.id)     // implicit, or wrap the mutation in withAnimation
```

- **The `.id()` must change, not just the string** — `.contentTransition` fires on identity change of the underlying view, so keying it to a stable string that happens not to change between two items (e.g. two movies that coincidentally share a genre chip) silently skips the cross-fade. Key to the item's own `id`, or to `text + current.id` for a per-field key (e.g. reusable "chip" subviews showing different fields of the same item).

---

## Native-Physics Scroll Stacks (invisible `UIScrollView` driver)

A `DragGesture`-driven stack (see Carousels & Paging above) always *approximates* momentum with a spring — it never quite matches real UIKit deceleration and rubber-banding. When a stack should feel like scrolling a native list (a stacked card/album deck that free-scrolls and decelerates like Music.app or Photos), drive it with a **real, invisible `UIScrollView`** instead, and let it push a plain `CGFloat` binding that a completely separate SwiftUI render stack reads to position its cards.

```swift
private struct ScrollDriver: UIViewRepresentable {
    @Binding var scrollOffset: CGFloat   // expressed in "items", not points
    let itemSpacing: CGFloat
    let maxOffset: CGFloat

    func makeUIView(context: Context) -> UIScrollView {
        let sv = UIScrollView()
        sv.delegate = context.coordinator
        sv.decelerationRate = .normal      // real momentum curve — not a spring
        sv.backgroundColor = .clear
        return sv
    }
    func updateUIView(_ sv: UIScrollView, context: Context) {
        sv.contentSize = CGSize(width: sv.bounds.width, height: maxOffset * itemSpacing + sv.bounds.height)
        guard !context.coordinator.isTracking, !context.coordinator.isDecelerating else { return }
        let target = scrollOffset * itemSpacing
        if abs(sv.contentOffset.y - target) > 0.5 {
            sv.setContentOffset(CGPoint(x: 0, y: target), animated: false)
        }
    }
    // Coordinator (UIScrollViewDelegate): scrollViewDidScroll → scrollOffset = contentOffset.y / itemSpacing
}
// The rendered card stack sits ABOVE the driver in a ZStack with .allowsHitTesting(false) on the
// stack — every touch lands on the invisible UIScrollView, which never draws any content of its own.
```

Non-obvious rules — each is a real trap:

- **The scroll view renders nothing.** Its `contentSize` exists purely to define the scrollable *range* and produce real deceleration/rubber-band physics; the actual cards are a totally separate layer reading `scrollOffset` as a plain float. This split — UIKit computes "where," SwiftUI (or a SceneKit-embedded card) draws "what" — is the core idea, and it generalizes to any custom-rendered stack that wants native scroll feel.
- **Gate every programmatic `setContentOffset` behind `!isTracking && !isDecelerating`.** Without this guard, the round-trip (driver → `@Binding` → `updateUIView` → `setContentOffset`) fights the user's own touch or native deceleration and stutters. Only force-sync the offset when the user isn't currently driving it themselves (e.g. an external "jump to index" call).
- **Settle-tick haptic — round to nearest, diff against the last-fired index**, not a raw pixel threshold: `let crossed = Int(floor(offset + 0.5)); if crossed != lastHapticIndex { tick(); lastHapticIndex = crossed }`. Fires `selectionChanged` exactly once per item boundary crossed in either direction — more robust than a fixed pixel threshold since it's expressed in "items," not points, and works the same regardless of `itemSpacing`.
- **Boundary/rubber-band haptic — fire once per overscroll excursion, not once per frame.** Gate with a `boundaryFired` flag set the first frame `contentOffset` passes either end, and reset it in `scrollViewWillBeginDragging` — otherwise a rigid impact fires on every `scrollViewDidScroll` tick while the user holds the view stretched past its limit (dozens of buzzes instead of one clean thud).
- **Windowed rendering — cull cards far from the visible window.** With many expensive per-item views (SceneKit, Metal, video), don't render the whole stack: `if abs(scrollProgress) <= renderWindow { cardView }` inside the `ForEach`. SwiftUI simply never builds the view for culled items — essential for SceneKit specifically, since each instance owns a real `SCNView` and render pass.

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

## Auto-Advancing Card Deck (swipe-or-timer, infinite wrap)

A Tinder-style deck (movie/song/product-of-the-day, story cards) where the front card **either** gets swiped away by the user **or** auto-advances on a timer, wraps around infinitely, and the next card must rise to the front *immediately* — without waiting for the discarded card to finish falling off-screen.

**Infinite wrap via modulo distance**, so `index` never needs bounds-checking:
```swift
private func slot(for i: Int) -> Int { (i - index + items.count) % items.count }   // 0 = front
```

**One `advance()` function, two callers.** Both the swipe gesture (past threshold) and a `.task` auto-advance timer call the same function — so the fall animation, haptic, and state mutation are defined once and can't drift apart between the manual and automatic paths:
```swift
.task {
    while !Task.isCancelled {
        try? await Task.sleep(for: .seconds(2.5))
        if flying == nil { advance(fromDrag: 0) }   // don't double-fire if a swipe is already mid-flight
    }
}
```

**The critical trap — decouple the exiting card from the stack it's leaving.** If the discarded card stays inside the same `ForEach` that renders the stack, bumping `index` (which reshuffles every remaining card's depth/offset) and animating that card's removal fight over the same elements — you get a stutter or a visible glitch. Instead render the falling card as an **independent overlay**, outside the stack loop, with its own state:

```swift
@State private var flying: Item? = nil
@State private var flyY: CGFloat = 0
@State private var flyRot: Double = 0

private func advance(fromDrag startY: CGFloat) {
    flying = items[index]; flyY = startY; flyRot = tilt(for: startY)
    withAnimation(.easeInOut(duration: 0.55)) { index = (index + 1) % items.count }   // stack reshuffles NOW, fast
    withAnimation(.easeIn(duration: 0.85)) { flyY = 1100; flyRot = 6 }                 // discarded card keeps falling, slower
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.87) { flying = nil }
}
// in the stack ForEach: skip any card whose id == flying?.id
// render the flying card separately: cardView(f).offset(y: flyY).rotationEffect(.degrees(flyRot)).zIndex(100)
```

- **Two different durations for two different things happening at once** — the stack's reshuffle (`0.55s`) is deliberately shorter than the card's fall (`0.85s`), so the next card is already settled into place while the discarded one is still visibly finishing its exit in the background. That's what makes it read as "the next one rose to the front," not "we waited for the old one to leave."
- `index` mutates the instant `advance()` is called — the fall animation never blocks it.
- **Progressive stepped stack offset** (tighter each level back) reads as a more physical, hand-stacked deck than a constant per-depth step:
```swift
private func stackOffset(_ depth: Int) -> CGFloat {
    var y: CGFloat = 0, step: CGFloat = 18
    for _ in 0..<depth { y += step; step = max(9, step - 3) }   // 18, 15, 12, 9, 9…
    return -y
}
```

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

## Liquid Glass Toasts & Status Banners

For Apple-style transient toasts/banners (success / error / warning / info / "copied") rendered in **Liquid Glass**, the glass only reads if there's **colorful content behind it** — put an image grid (or any vivid backdrop) behind the toast layer so the refraction is visible; a flat dark/light background hides the effect.

```swift
// neutral glass = maximum refraction; status color lives in the icon, NOT the glass tint
.background(glassBackground(Capsule(), tint: nil))   // .glassEffect(.regular) / .ultraThinMaterial fallback
```

Non-obvious rules — each one is a real trap:

- **A backdrop taller than the screen must go in `.background`, never as a ZStack sibling.** *(headline)* An image grid / `LazyVGrid` whose intrinsic height exceeds the screen, placed as a ZStack sibling, **inflates the container's height** — so a `Spacer()`-anchored control (e.g. a bottom button) is pushed *off-screen* and silently disappears. Move the backdrop into `.background { grid }.clipped()` so it fills the screen but **cannot drive the foreground's layout**; the foreground then lays out against the true viewport. Corollary: don't stack an opaque `Color` *in front of* a `.background` layer — it hides it.
- **Keep every toast's glass identical; put status in the icon, not the surface.** Tinting the glass per status (`.glassEffect(.regular.tint(statusColor))`) makes that toast read as a near-**solid colored fill** while the neutral ones refract, so they look inconsistent. Use neutral `.regular` glass for *all* toast surfaces (capsule and card alike) and carry the status color in the SF Symbol / a small tinted icon well. This is the **opposite** of the *tinted-glass-for-destructive-buttons* rule — toasts want consistency, controls want signalling.
- **Toast position respects the safe area for free.** An `.overlay(alignment: .top) { toast }` aligns to the safe-area top, so a small `.padding(.top, 10)` drops it just below the Dynamic Island. A *fixed* top pad (`.padding(.top, 70)`) lands under the notch whenever the view ignores the top safe area — don't hardcode it.
- **Auto-dismiss with a token, not a bare delay.** On show, set `toast = new`; schedule dismissal after ~2.5s but only clear it `if toast?.id == new.id`, so a newer toast isn't cut short. Slide in/out with `.transition(.move(edge: .top).combined(with: .opacity))` inside `withAnimation`.
- **Shapes:** `Capsule` for single-line statuses, a `RoundedRectangle` card (icon well + title + subtitle) for richer ones — both on the **same neutral glass**.

**Icon-only Liquid Glass action button (cycling trigger)** — an Apple-native glass button that morphs its icon on each tap:

```swift
Button(action: trigger) {
    Image(systemName: kind.icon)
        .contentTransition(.symbolEffect(.replace))   // morph to the next action
        .frame(width: 72, height: 72)
}
.buttonStyle(.glass)            // iOS 26 — material-circle fallback below
.buttonBorderShape(.circle)
.tint(kind.tint ?? .white)
// trigger(): show(current); withAnimation { index = (index + 1) % count }
```

- Just the icon — let the glass and its refraction be the whole button; the `.replace` transition (driven **inside `withAnimation`**) is the headline interaction. Don't add a text label or a custom ring.
- **Haptic ladder for statuses:** `notificationSuccess / Warning / Error` for those kinds, `lightImpact` for neutral info/copied.

---

## Stacked Deck & In-Place Reorder

A set of cards that lives **inside** a row that the parent already made
draggable / long-pressable (a list item with `.draggable` or a `.contextMenu`).
Two traps, one archetype.

**Don't nest drag in drag.** A child `.draggable` inside an already
drag/long-press-enabled parent makes nested recognizers fight for the same touch
— the parent's long-press and the child's drag both arm on the same gesture arc,
so neither feels reliable. Scoping the child drag to a small grip handle does
**not** fix it; the parent's long-press still fires from that spot.

- **Reorder with discrete tap controls, not drag.** Per-row up/down chevron
  buttons (▲ disabled on the first row, ▼ on the last) that move an item one
  slot. Buttons fire on *tap*, which never competes with a long-press, so reorder
  works inside any card. This is the same fallback Apple uses where drag is
  unavailable.
- **Persist positionally in place.** Rewrite the i-th slot of the underlying data
  (e.g. reorder URL tokens within the snippet text, leaving interleaved prose
  exactly where it was; no-op unless counts match so text can't be mangled) so
  the new order survives reload.

**Notification-deck archetype** (collapsed peek-stack → fan open): lay every card
in a `ZStack(alignment: .top)` and switch each card's geometry on one `isExpanded`
spring.

```swift
let depth = CGFloat(min(index, maxPeeks))            // 0,1,2 — only top few peek
card
    .frame(height: rowHeight)
    .scaleEffect(isExpanded ? 1 : 1 - depth * 0.05, anchor: .top)   // peeks shrink
    .brightness(isExpanded ? 0 : -Double(depth) * 0.03)             // …and dim
    .offset(y: isExpanded ? CGFloat(index) * rowStride              // fanned list
                          : depth * peekStep)                       // stacked peek
    .opacity(!isExpanded && index > maxPeeks ? 0 : 1)
    .zIndex(Double(count - index))
```

- Collapsed, the whole deck is **one tap target** that expands (overlay a clear
  `Button`); the cards underneath stay `allowsHitTesting(false)` until open.
- A glass count pill (`rectangle.stack` + N) overlays the collapsed top card.
- Container height is computed per state: `count * rowHeight + (count-1) * spacing`
  expanded vs `rowHeight + min(count-1, maxPeeks) * peekStep` collapsed.
- `mediumImpact` on expand/collapse; `selection` on each move.

---

## Metal Shaders (`.colorEffect` / stitchable)

For premium GPU surfaces plain SwiftUI can't reach — **liquid chrome you can poke, holographic foil, plasma, molten metal** — write a real Metal shader and apply it with `.colorEffect`. This is a genuine `[[ stitchable ]]` shader in a separate `.metal` file, **not** a Canvas look-alike. (legendary-Animo already ships ~11 of these, so it's an established pattern — check for existing `.metal` files first.)

```metal
// LiquidMetal.metal
#include <metal_stdlib>
#include <SwiftUI/SwiftUI_Metal.h>
using namespace metal;

[[ stitchable ]]
half4 liquidMetal(float2 pos, half4 color, float2 res, float time, float4 r0 /*…*/) {
    float2 uv = pos / res;
    // FBM height field → normal → moving light (diffuse + specular + fresnel)
    return half4(half3(col), 1.0h);
}
```
```swift
Rectangle().foregroundStyle(.black)
    .colorEffect(ShaderLibrary.liquidMetal(.float2(size), .float(time), .float4(x,y,startT,amp)))
```

Non-obvious rules — each one is a real trap:

- **Register the `.metal` in the pbxproj *Sources* phase as `sourcecode.metal`.** *(headline)* Adding the file is not enough — if it isn't in Compile Sources it never lands in `default.metallib`, so `ShaderLibrary.<name>` silently renders **black** (no crash, no error). The `[[ stitchable ]]` function name must match the `ShaderLibrary.<name>` call exactly.
- **The first two params are supplied automatically.** A `.colorEffect` shader is `half4 fn(float2 pos, half4 color, <your uniforms…>)` — SwiftUI fills `pos`/`color`; your Swift arg list starts at the first uniform. (`.distortionEffect` warps geometry; `.layerEffect` can sample neighbours — needs `<SwiftUI/SwiftUI_Metal.h>`.)
- **Drive it from a `TimelineView(.animation)` clock, never a spring.** A shader field is a continuous loop; pass `time = ctx.date.timeIntervalSince(start)` as a `float` uniform and re-issue the `.colorEffect` each tick. (Same rule as Loading Indicator / Canvas loaders.)
- **Touch data has no float-array-of-structs — pack a fixed `float4` ring-buffer.** SwiftUI `Shader.Argument` only has `.float/.float2/.float3/.float4/.color/.image/.floatArray`. For N touch ripples keep a Swift ring-buffer capped at, say, 4 and pass 4 fixed `.float4(x, y, startTime, amp)` slots (pad empties with amp 0). Capping to the last few while dragging gives a trailing **liquid wake** for free.
- **Ripples perturb the height field, not the color.** Add each decaying ring (`sin(d·k − age·ω)·exp(−age)·exp(−d)`) into the surface height *before* computing the normal, so the *reflection* bends where you poke — that's what sells "liquid", not a color splash.
- **Gate `iOS 17+`** (`.colorEffect` is 17.0) with a non-shader fallback (e.g. an `AngularGradient` sweep) so it still builds on 16.
- **Verification trap:** a shader can compile+link and still render black (name mismatch, missing Sources entry, wrong arg order). **Build and look** — `CompileMetalFile` + `MetalLink` in the log prove compilation, but only a screenshot proves it renders.

**Content taste:** lean into the double meaning of *Metal* — molten chrome/mercury/gold/oil-slick reads as premium and unexpected. Keep alloys as tint-pair uniforms so one shader serves many palettes; carry iridescence in the fresnel, not a flat overlay.

---

## SceneKit 3D Object Showcase

For a **real 3D object** (an album cover, a book, a boxed product) that tilts/rotates in response to scroll or touch — not a fake `rotation3DEffect` pseudo-3D — embed an `SCNScene` via `UIViewRepresentable` and drive it from SwiftUI state.

### Truly transparent SceneKit embed

`SCNView` defaults to an **opaque white background** — invisible over a SwiftUI backdrop unless cleared in all four places:

```swift
scene.background.contents = nil        // the SCNScene itself
view.isOpaque = false
view.backgroundColor = .clear
view.layer.isOpaque = false
```
Missing any one of these still shows a white (or black, post-clear-but-pre-layer) rectangle behind the object.

### Drive node transforms from SwiftUI state without SceneKit's implicit animation fighting you

SceneKit **implicitly animates** node property changes by default. If a tilt/rotation should track a continuously-driven value (scroll offset, drag), that implicit animation makes it lag behind and rubber-band against your own driver. Wrap every driven update in a transaction with actions disabled:

```swift
SCNTransaction.begin()
SCNTransaction.disableActions = true
node.eulerAngles = SCNVector3Make(tiltX * .pi / 180, tiltY * .pi / 180, 0)
SCNTransaction.commit()
```
The value then updates **exactly** on your schedule (every `updateUIView`, every scroll tick) — SwiftUI/UIKit owns the animation curve, not SceneKit's own defaults.

### Boxed-object recipe (6-face `SCNBox`, one face as a rendered label)

An `SCNBox`'s six materials, in `SCNBox.materials` order (+Z front, -Z back, +X right, -X left, +Y top, -Y bottom), build a "sleeve" object with a real spine: front face = artwork, spine face = a **dynamically rendered `UIImage`** (title + subtitle drawn via Core Graphics onto a colored background), remaining faces = a solid edge color:

```swift
box.materials = [coverFront, edgeColor, coverRight, edgeColor, edgeColor, labelEdge]
```
- Image faces use `.lightingModel = .blinn` (should shade under the lights); solid-color faces use `.lightingModel = .constant` (flat — doesn't need lighting math, cheaper).
- The spine label is a plain `UIGraphicsImageRenderer` draw — no SceneKit text API needed. Render it **once** (not per frame) and hand the resulting `UIImage` to `.diffuse.contents`.

### Lighting/camera rig — reusable default for "object floating in space"

One directional **key** light + one **ambient** fill is enough for a premium single-object showcase — no need for a full 3-point rig:
```swift
keyLight.type = .directional; keyLight.intensity = 900                                 // tune 700–1000
keyLight.eulerAngles = SCNVector3Make(-40 * .pi/180, 25 * .pi/180, 0)                   // top-front-left
ambientLight.type = .ambient; ambientLight.color = UIColor(white: 0.30, alpha: 1)       // fill so shadow side isn't pitch black
camera.fieldOfView = 46; camera.position = SCNVector3Make(0, 0.3, 5.0)
```

### Dual-mode component: standalone vs. embedded

The same object view should work both as a full-screen showcase (opaque black backdrop, full antialiasing) and as one of many repeated instances inside a scrolling stack (transparent, cheaper antialiasing since several render at once):
```swift
var embedded: Bool = false
// standalone: ZStack { Color.black.ignoresSafeArea(); sceneView }
// embedded:   sceneView alone, background stays .clear
view.antialiasingMode = embedded ? .multisampling2X : .multisampling4X   // cheaper AA when many instances render together
```

**Verification trap:** a SceneKit view can compile and run while still showing a plain white/black rectangle if any transparency step above is skipped, or a flat gray box if a texture failed to load (`UIImage(named:)` returning `nil` fails silently). Screenshot-verify the actual object renders with correct art and lighting — don't just confirm the view exists in the hierarchy.

---

## Deriving Accent Colors from Artwork

When a screen should tint itself (an object's spine face, a background scrim, a button accent) from the dominant color of an image rather than a hardcoded palette value, sample a **downscaled** copy and weight by saturation + brightness — not raw pixel frequency, which usually just picks whatever's most common (often a boring background gray or black):

```swift
let sampleSize = 40   // downscale first — sampling a full-res image per pixel is wasteful and noisy
context.draw(cgImage, in: CGRect(x: 0, y: 0, width: sampleSize, height: sampleSize))
// per pixel: visibility = saturation * 0.7 + brightness * 0.3
// bucket by quantized color, accumulate weight * (r,g,b), skip alpha < 0.5
// winner = bucket with highest total weight; divide its accumulated color by its weight
```

- **Weight by saturation + brightness, not pixel count.** A plain "most frequent color" pass on a poster/cover usually returns the boring gray/black background fill. Weighting toward saturated, mid-brightness pixels finds the color a human would actually call "the color of this image."
- **Reject near-white/near-black outliers explicitly** (skip pixels with `brightness` outside `0.08–0.95`) — otherwise a mostly-white or mostly-black image just returns white/black as its "dominant" color, which is true but useless as an accent.
- **Darken a near-white winner** (`luminance > 0.85 → multiply channels by 0.75`) — even after weighting, a pastel-heavy image can still win with a color that reads as a blank white panel when filled as a solid background; scaling it down keeps it visibly a color rather than a void.
- **Contrasting text color — use perceptual luminance, not a channel average:** `0.299·r + 0.587·g + 0.114·b > 0.55 → black text, else white`. Green reads brighter to the eye than red or blue at the same channel value, so a plain `(r+g+b)/3` average picks the wrong text color on saturated pure-blue or pure-green backgrounds.

---

## Live Activities & the Real Dynamic Island (ActivityKit + WidgetKit)

An in-app view can *mimic* the Dynamic Island (morph, glass, gestures) — but to render in the **actual** Dynamic Island + Lock Screen you need **ActivityKit + a Widget Extension**, and the rules are nothing like an in-app SwiftUI view. Build the mimic for the micro-interaction; build a Live Activity when it must live in the real island.

- **It requires a separate Widget Extension target — not a view.** Declare `struct XAttributes: ActivityAttributes` (with a `ContentState`), add an `ActivityConfiguration(for:)` + `DynamicIsland { }` DSL in a **WidgetKit extension**, set `NSSupportsLiveActivities = true` in the app's Info.plist, and the app calls `Activity.request / update / end`. There's no CLI to scaffold the target — use Xcode's *File → New → Target → Widget Extension* (check "Include Live Activity").
- **WidgetKit ≠ in-app SwiftUI: no gestures, no continuous animation, no custom springs.** *(headline)* The compact↔expanded **morph is the system's** (genuine Liquid Glass on iOS 26) — you only supply each region's content. Interactivity is **App-Intent Buttons only** (`Button(intent:)` backed by a `LiveActivityIntent`), never `DragGesture`/`onTapGesture`. A scrubber can't be dragged; a progress bar only advances on a *pushed* state update. No `TimelineView` 60 fps loop. So the drag / rubber-band / live-equalizer micro-interaction stays **in-app**; the island is a declarative status display.
- **Artwork memory budget is the #1 blank-island gotcha.** *(headline)* A Live Activity extension has a hard ~30 MB memory limit. A full-res photo (`4000×6000` ≈ 96 MB decoded) **silently fails to render → grey placeholder box** — no crash, no error. Ship artwork at ~**400×400** (it displays tiny). This is the single most common "why is my island art blank."
- **Expanded regions clip — keep them compact.** The `DynamicIslandExpandedRegion` set (leading / trailing / center / bottom) has a max height; art + title + progress + times + a big transport row overflows and clips at the bottom. Use small control fonts (`.title2` play, `.body` skips), tight spacing, and put elapsed / −remaining **under** the progress bar, not as a floating trailing number.
- **Shared `ActivityAttributes` must belong to BOTH targets.** The attributes / `ContentState` file is compiled into the app (which requests/updates) *and* the widget (which renders) — one `PBXFileReference`, a build-file entry in each target's Sources phase (or a synchronized-folder membership). It's compiled into each module, so **no `import`** of the other.
- **Modern Xcode widget targets are file-system-synchronized** (`PBXFileSystemSynchronizedRootGroup`): a file dropped into the extension's folder auto-joins the target and deleting the template samples auto-removes them — only cross-target shared files need explicit wiring.
- **Two-way island ↔ app sync.** The island's App-Intent buttons mutate the running activity directly (`Activity<Attrs>.activities.first` → `update(...)`); the app observes `activity.contentUpdates` to reflect island-driven changes back into its own UI, keeping `isPlaying` / progress consistent on both sides.
- **Content taste:** a real, *small* album cover on the black pill reads premium; a gradient + SF-Symbol placeholder reads as "no artwork."

---

## SceneKit Metallic Badge — flat-face lighting, coin-flip settle, engraved back

A spinnable metal achievement badge (shield / coin / star): extruded `SCNShape` bezel + colored enamel + raised emblem, PBR metal reflecting a code-generated environment gradient. Distinct from the boxed-object showcase — these traps are specific to *flat, front-facing metal*.

- **Flat metal faces mirror the ENVIRONMENT, not a light.** *(headline)* A pure-metal (`metalness 1`) flat face pointing at the camera reflects the env map's camera-facing region — usually dark — so it renders **black on a dark background**. A directional key light adds *no* diffuse to pure metal, so it can't rescue it. Fixes: use **satin metal (`metalness ≈ 0.7`, `roughness ≈ 0.34`) for flat faces** so the tint reads; reserve mirror-chrome (`roughness ≈ 0.15`) for **beveled / curved** surfaces (they catch the bright top of the env). A beveled crest looks great head-on; a flat coin does not without this.
- **Add a front fill light for static / head-on metal.** Even satin metal at rest looks dull with only top + rim lights. A low directional light from ~straight-front — plus a small resting **lean** so the face tilts toward the brighter upper env — keeps the front lit with a central specular. Pair with a high-contrast env gradient (bright top → near-black bottom) and a raised `lightingEnvironment.intensity` for punchy reflections.
- **Coin-flip settle (no auto-spin).** Drive one `spinAngle` from the render loop; when not dragging, spring it toward the **nearest multiple of π, recomputed every frame** — `target = round(spinAngle / .pi) * .pi`. A flick seeds `spinVel` from `DragGesture.value.velocity`; because the target re-picks each frame, momentum carries the badge face-to-face and it lands on the nearest face (heads/tails) with a slight wobble. Low stiffness (~50) lets a flick pass through midpoints instead of snapping back.
- **The engraved back plane must be occluded from the front.** A double-sided textured plane behind the body bleeds through the front (peeks past a tapered silhouette, or shows through dark enamel). Push it **well behind the opaque body** (past the back face in `z`) and keep it **inside the body's silhouette** (smaller than the widest shape) so the body occludes it head-on; it only appears once the badge flips. And **do not pre-mirror the back text** — the parent's `y = π` flip already presents the plane's *front* to the camera, so a manual horizontal flip double-mirrors it (reads backwards).
- **Concentric-ring coin (Apple Move-award motif):** metal base `SCNCylinder` + colored recessed face + raised `SCNTorus` ring grooves + a center boss, each node rotated `π/2` about X to lay the coin flat facing the camera.
- **Fidelity ceiling — procedural vs `.usdz`.** Extruded `SCNShape` + code-gen env *approximates* the look, but Apple's badges are authored, texture-baked `.usdz` with real HDRI reflections. For pixel-parity, load a `.usdz` (Reality Composer Pro / Blender export) instead of building geometry — procedural is the "good, self-contained" tier, not the parity tier.

---

## Ring Gauges — trim fill + cap-dot geometry

For Apple Fitness-style activity rings (concentric gradient rings that fill):

- **A leading cap dot on a `Circle().stroke` ring orbits at `side/2`, not `(side − lineWidth)/2`.** *(headline)* `Circle().stroke(lineWidth:)` centers the stroke on the path (radius `side/2`, bleeding `lineWidth/2` outside the frame). A cap dot placed at `(side − lineWidth)/2` sits half a band-width *inside* the ring. Orbit it at `side/2` — `.offset(y: -side/2)` then `.rotationEffect(.degrees(progress * 360))` around the ZStack center — so it rides *on* the band.
- **Staggered spring trim-fill.** Fill each ring `Circle().trim(from: 0, to: min(progress, 1))` from 0 with `withAnimation(.spring).delay(0.08 * i)` per ring; roll the stat numbers with `.contentTransition(.numericText(value:))`.
- **Overachieve wrap-with-shadow (the >100% cue).** For `progress > 1`, draw a second `trim(0, progress − 1)` arc on top and place a **shadowed cap dot** rotated to `progress * 360°` so it overlaps the ring start — the depth cue that reads as "you beat the goal."
- **Vivid ring gradients** (pink→red move, green exercise, cyan→blue stand) are the one place the "never pure `.red/.green/.blue` on dark" rule bends — these are *tuned* gradients, not muddy system colors.

---

## Liquid Glass Tab ⇄ Search Morph

The iOS 26 signature: a floating glass tab bar whose search button morphs into a full-width search field.

- **It's fuse/separate of TWO `glassEffectID`s, not one element resizing.** *(headline)* Give the tab-bar capsule one id (`"bar"`) and the search circle another (`"search"`); on toggle, **remove the bar** (it melts away) while the **search element grows from circle → full-width field** (same `"search"` id). `GlassEffectContainer` renders the metaball melt. A single resizing element can't express "one thing leaves while another expands."
- **Delay keyboard focus until after the morph settles.** Auto-focusing the field mid-morph shoots the keyboard up and steals the transition — schedule `@FocusState = true` *after* the morph spring (e.g. `+0.6s` for a slow morph) so the circle→field morph plays first.
- **Slow the morph so it's felt** (spring `response ≈ 0.95`), and respect the fusion-threshold rule (`layoutGap > containerSpacing + 6`) so the bar and search circle don't bleed into each other at rest.
- **Proper tab-bar feel:** icons only, each `.frame(maxWidth: .infinity)` so they spread evenly (no dead space); show selection with a soft circle behind the icon, not a text label.

---

## Output (Create mode)

Stream these progress lines one by one:

```
⚙️  swiftui-microinteractions v1.20.0
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
