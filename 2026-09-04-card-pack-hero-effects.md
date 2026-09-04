# 2026-09-04 — Card Pack Reveal + Hero Power Effects

Learnings from building **FootballCardPack** (sealed foil pack with swipe-to-tear reveal, 3D card flip, staggered stat rings, card browser with live drag) and **HeroPowerShowcase** (per-hero `TimelineView + Canvas` power effects).

---

## Public learnings (landed in SKILL.md v1.24.0)

### Counter-flip for cumulative `rotation3DEffect`

A Y-axis `rotation3DEffect` mirrors all inner content at every odd multiple of 180 degrees. When a card accumulates rotation across sequential flips (180 degrees per "next card"), text and images read backwards on every other reveal.

**Naive:** rotate the whole ZStack and hope the content stays readable.

**Correct:** compute `halfTurns = Int(round(angle / 180))` and apply `.scaleEffect(x: halfTurns % 2 != 0 ? -1 : 1)` on the inner content *before* the rotation3DEffect. The two mirrors cancel and content always reads correctly.

### `SeededRNG` for deterministic Canvas particles

`CGFloat.random()` inside a `Canvas` closure produces different values every frame, making particles flicker. A simple xorshift struct (`SeededRNG`) with a `unit() -> CGFloat` method keeps positions stable frame-to-frame. Change the seed on *events* (new lightning strike, new impact), not per frame.

Same principle applies outside Canvas: never call `.random()` inside a `@ViewBuilder` body. Pre-compute random arrays via a plain function and store as a `let`.

### Page indicator sliding-indicator variant

The existing variable-width capsule dots work well when all dots share one style. When each dot has a *different* active color (e.g. tier-colored: gold/silver/bronze), the capsule must land dead-center on the target dot. `matchedGeometryEffect(id:in:)` on a single sliding capsule handles this — SwiftUI interpolates position exactly. Manual offset math drifts because `HStack` spacing interacts unpredictably with variable capsule widths.

### `PBXFileSystemSynchronizedRootGroup` gate

Projects with `objectVersion >= 77` (Xcode 16+) use file-system-synchronized groups. Files placed in the source directory compile automatically with no `.pbxproj` edits. The Xcode registration script must check for this before attempting a manual edit, or it corrupts a synchronized project.

### Stable `ForEach` identity

`ForEach(items.indices, id: \.self)` uses index-based identity. When the array reorders, SwiftUI tears down and rebuilds views — breaking in-flight animations. `ForEach(Array(items.enumerated()), id: \.element.id)` ties identity to the element, so reorders animate correctly.

---

## Pro-routed learnings (not in public SKILL.md)

The following patterns were built but routed to Pro as archetype teardowns:

- **Swipe-to-reveal with masked image**: progressive `.mask` driven by drag gesture to unveil a hidden image top-to-bottom; bidirectional tracking; phase gate at progress < 0.01.
- **Light/dark `PackTheme` struct**: art-directed dual-palette threading through subviews; 3D raised button in light vs glass-outline in dark; tier badge style switching.
- **Per-hero Canvas power effects**: branching lightning tree (glow pass + hot-core pass), HUD rings (hex mesh + reactor + targeting), smash shockwave (elliptical rings + seeded cracks + debris with gravity + screen shake), web mesh (radial spokes + scalloped rings + corner strands + hexagonal sense rings), infinity stones orbit (comet trails via past-position sampling + energy links + snap dust wave).
- **Staggered stat-ring fill with DispatchQueue counting**: recursive tick driving displayed integer from 0 to target in sync with `.trim` animation; `.contentTransition(.numericText(countsDown: false))` for per-digit roll-up. (Differs from Ring Gauges' spring approach — use `easeOut` + manual counter when the fill duration is long and the number must track the arc precisely; use Ring Gauges' spring when the fill is short and `.numericText(value:)` suffices.)
