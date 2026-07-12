# Changelog

All notable changes to `swiftui-microinteractions` are documented here.

Format: `[version] — date — summary`

---

## [1.19.0] — 2026-07-12

Live Activities & the Real Dynamic Island (ActivityKit + WidgetKit): the in-app island is a look-alike — the real island needs a Widget Extension; WidgetKit has no gestures / continuous animation / custom springs (interactivity = App-Intent buttons only); the ~30 MB artwork memory budget → grey-box gotcha (ship ~400×400); expanded-region clipping; shared `ActivityAttributes` two-target membership; island↔app `contentUpdates` sync.

---

## [1.18.0] — 2026-07-06

Learnings from building a Song App (a 3D SceneKit album-cover object driven by a native-physics vertical stack carousel) — the skill's first real 3D/SceneKit and UIKit-scroll-physics patterns.

- **SceneKit 3D Object Showcase section (new)**: embed a real `SCNScene` via `UIViewRepresentable` for objects that tilt/rotate in true 3D (not `rotation3DEffect`). Truly transparent embed needs **four** places cleared (`scene.background.contents = nil` plus `isOpaque`/`backgroundColor`/`layer.isOpaque` on the view) — missing any one leaves a white/black rectangle. Driving a node's transform from continuously-changing SwiftUI state (scroll offset, drag) needs `SCNTransaction.disableActions = true`, or SceneKit's own implicit animation fights and lags behind the driver. A 6-face `SCNBox` can act as a "sleeve" object — front face art, one face a **dynamically rendered label image** (Core Graphics, drawn once), rest solid color. One directional key light + one ambient fill is a reusable default lighting rig. A dual `embedded: Bool` mode (opaque standalone vs. transparent + cheaper antialiasing when many instances render together) lets one component serve both a hero showcase and a repeated stack item.
- **Deriving Accent Colors from Artwork section (new)**: sample a downscaled copy of an image and weight pixels by saturation+brightness (not raw frequency) to find a representative accent color — plain "most frequent" just returns the boring background fill. Reject near-white/near-black outliers explicitly; darken a near-white winner so it doesn't read as a blank panel. Contrasting text color uses perceptual luminance (`0.299r + 0.587g + 0.114b`), not a channel average, since green reads brighter to the eye than red/blue.
- **Native-Physics Scroll Stacks section (new)**: for a stack that should feel like scrolling a native list (real momentum/rubber-banding, not a `DragGesture` spring approximation), drive it with an invisible, content-less `UIScrollView` whose `contentOffset` pushes a plain `CGFloat` binding — a completely separate SwiftUI layer reads that binding to position its own cards. Traps: gate programmatic `setContentOffset` behind `!isTracking && !isDecelerating` or the binding round-trip fights the user's touch; fire a settle-tick haptic by rounding to nearest index and diffing against the last-fired index (robust regardless of item spacing); gate the boundary/rubber-band haptic with a `boundaryFired` flag reset on next drag so it fires once per excursion, not once per frame; cull far-off items with a `renderWindow` check — essential when each item is an expensive per-instance render (SceneKit, Metal, video).
- **Asymmetric distance-response curves (extends Coverflow)**: nothing requires a continuous fan/stack transform to be symmetric — branch the degrees-per-step (or scale/opacity falloff) on the *sign* of the distance, not just its magnitude, when the two directions should read as physically different (cards already passed vs. cards still ahead).
- **New Archetype Catalog row**: `3D Object Showcase` — SceneKit rig physics, `SCNTransaction.disableActions` for driven properties, never a spring.
- Version output bumped to `⚙️ swiftui-microinteractions v1.18.0`

---

## [1.17.0] — 2026-07-06

Learnings from building NowPlayingView (a Liquid Glass "Now Playing" screen with an auto-advancing, infinite-wrap poster deck that's also swipe-to-dismiss, plus a content-driven glass tab bar).

- **Auto-Advancing Card Deck section (new)**: the Tinder-style deck archetype — infinite wrap via `(i - index + count) % count`, one `advance()` function called by both the swipe gesture and a `.task` timer so the two paths can't drift apart, and the critical trap: render the discarded/falling card as an **independent overlay outside the stack's `ForEach`**, not as a removal inside it, so bumping `index` reshuffles the stack immediately instead of fighting the exit animation. Two deliberately different durations (fast reshuffle, slower fall) sell "the next one rose to the front" rather than "we waited." Includes a progressive stepped-offset formula (18, 15, 12, 9…) as a more physical alternative to a constant per-depth step.
- **Gate-and-rearm threshold haptics (new, under Haptics)**: a drag that crosses a threshold, retreats, and crosses again should buzz on every crossing — gate on entry, rearm on exit (`didCrossThreshold` reset when the drag falls back below the line), not a fire-once-ever boolean.
- **Liquid Glass moving-content trap (new)**: never put a live `.glassEffect()` on a chip attached to a view that's being actively dragged/animated — glass resamples its backdrop continuously and visibly "grows/pulses" on fast-moving elements. Use a static translucent color fill for anything riding on moving content; reserve real glass for static or slow-moving elements.
- **`.regular` vs `.clear` glass style (new)**: `.clear` is more transparent than `.regular` — reach for it on a bar/control over especially rich or busy content (a poster/photo backdrop) where `.regular` would over-flatten the art.
- **Expanding tab bar variant (new, under Tab Bar Patterns)**: a content-driven alternative to the sliding-indicator bar — only the selected tab shows its label, so the glass capsule autosizes around content with no indicator math at all; right for a floating bar over content, not a fixed-width full-screen bar.
- **Image-colored ambient shadow (new, under Style Rules)**: duplicate the card's own artwork behind it, heavily blurred/dimmed/offset, instead of a plain black `.shadow` — the halo reads as tinted by the actual content instead of generic weight. Reserve for hero cards, since it's a duplicate image render.
- **Mask-vs-scrim fade-out (extends the selection-synced backdrop pattern)**: a `LinearGradient` scrim dims a photo so text stays legible *on top of it*; a `.mask` with the same gradient shape instead *reveals* a solid color behind the image for a hard, clean handoff to a card below — pick by what's underneath the backdrop, not by habit.
- Version output bumped to `⚙️ swiftui-microinteractions v1.17.0`

---

## [1.16.0] — 2026-07-06

Learnings from building MoviePosterSwipeCarousel (a velocity-aware movie poster carousel with a coverflow-style continuous fan, a selection-synced blurred backdrop, and variable-width page dots).

- **Coverflow / continuous depth fan (new, under Carousels & Paging)**: replaces the two-state (`selected`/`not`) fan with one **signed, drag-blended distance** (`(i - selectedIndex) - dragOffset/cardWidth`) driving scale, rotation (capped ±14°), y-offset, opacity, and blur-by-distance together, so every card — not just the immediate neighbor — recedes continuously and updates *while the finger is still dragging*, not just on release. Blur-by-distance is called out as a cheap depth-of-field that sells "depth" far more than scale/opacity alone.
- **Selection-synced immersive backdrop (new)**: a full-bleed blurred+saturated background that cross-fades to match the selected carousel item. Stack all candidate images and cross-fade `opacity` — never swap which `Image` is in the tree, or the cross-fade can't animate. `.blur(radius:opaque: true)` avoids the translucent-edge artifact plain `.blur` produces at large radii; apply blur/saturation once to the whole stack, not per-image.
- **Page indicator dots (new)**: a lightweight capsule-dot position indicator (active dot stretches 7pt → 22pt) as a simpler alternative to the Tab Bar sliding indicator when there's no tab content to align to.
- **Detail text swap on selection (new)**: `.contentTransition(.opacity)` keyed to a changing `.id()` as a lightweight alternative to `.numericText` for non-numeric text (titles, metadata) that should cross-fade with a selection change — with the trap that the `.id()` must actually change per item, not just the displayed string.
- **Gold/rating color added to Style Rules**: `Color(red: 1.0, green: 0.84, blue: 0.35)` for star ratings and similar accents.
- **Gloss overlay recipe added to Style Rules**: a diagonal `.white.opacity(0.18) → .clear` gradient with `.blendMode(.softLight)` over a photographic card reads as premium glass sheen without washing out the photo (softLight, not screen, is what keeps it subtle).
- Version output bumped to `⚙️ swiftui-microinteractions v1.16.0`

---

## [1.15.0] — 2026-07-01

Added a **Metal Shaders (`.colorEffect` / stitchable)** section — learnings from building a poke-able molten liquid-metal surface as a real GPU shader (not a Canvas look-alike).

- **New Metal Shaders section**: write a `[[ stitchable ]]` shader in a `.metal` file and apply it with `.colorEffect`; drive it from a `TimelineView` clock (never a spring); gate `iOS 17+` with a non-shader fallback.
- **Register the `.metal` in the pbxproj *Sources* phase as `sourcecode.metal`** — the silent-black trap: if it isn't compiled into `default.metallib`, `ShaderLibrary.<name>` renders black with no error. The stitchable function name must match the `ShaderLibrary.<name>` call.
- **Pack touch data as a fixed `float4` ring-buffer** (SwiftUI has no float-array-of-structs); ripples perturb the *height field* before the normal is computed, so the reflection bends where you poke — that's what sells "liquid".
- **Designer vocab → Metal**: "liquid chrome / molten metal / mercury", "holographic / iridescent / oil-slick", "plasma / lava / aurora" now route to a `.colorEffect` shader; added a **Metal Shader** archetype row.
- Version output bumped to `⚙️ swiftui-microinteractions v1.15.0`

---

## [1.14.0] — 2026-06-26

Strengthened the **iOS 26 Liquid Glass** section with a "use the genuine effect, always" rule.

- **When a prompt says "Liquid Glass", use the real `.glassEffect`** — never fake it with a near-opaque material/tint as the primary surface (a `Capsule().fill(.ultraThinMaterial)` or heavy white-tinted glass reads as a flat chip, not glass). `.ultraThinMaterial` is the `< iOS 26` fallback only.
- **Put busy, colorful content behind the glass** (image grid / photo / vivid gradient) so the refraction reads — a flat solid or pale background hides Liquid Glass and is the #1 reason a "glass" build looks wrong. For a standalone showcase, an image-tile grid backdrop is the safest way to make it pop.
- Version output bumped to `⚙️ swiftui-microinteractions v1.14.0`

---

## [1.13.0] — 2026-06-26

Learnings from building a multi-link "stacked deck" for snippet cards (several links collapse into a notification-style stack that fans open; cards reorder in place), where the reorder had to live inside a card the parent list already made draggable / long-pressable.

- **Stacked Deck & In-Place Reorder section (new)**: the notification-deck archetype (collapsed peek-stack → one-spring fan-out, peeks shrink + dim via `scaleEffect`/`brightness`, a glass count pill, the whole collapsed deck is one tap target) plus the reorder rule.
- **Don't nest drag in drag**: a child `.draggable` inside an already drag/long-press-enabled parent makes nested recognizers fight for the same touch — and scoping the child drag to a grip handle does **not** fix it (the parent's long-press still arms there). Reorder with **discrete tap controls** (▲▼ buttons, fire on tap) instead — taps never compete with a long-press.
- **Persist positionally in place**: rewrite the i-th data slot (e.g. reorder URL tokens within the text, leaving interleaved prose put; no-op unless counts match) so order survives reload.
- Version output bumped to `⚙️ swiftui-microinteractions v1.13.0`

---

## [1.12.0] — 2026-06-24

Learnings from building LiquidGlassToastsView (Apple-style Liquid Glass toasts over an image grid, triggered by an icon-only glass button that cycles statuses via the SF Symbol replace transition).

- **Liquid Glass Toasts & Status Banners section (new)**: how to render transient toasts/banners in Liquid Glass so the effect actually reads, plus the traps. **A backdrop taller than the screen must live in `.background`, never as a ZStack sibling** — an image grid / `LazyVGrid` whose intrinsic height exceeds the screen inflates the container, pushing `Spacer()`-anchored controls (a bottom button) off-screen; move it to `.background { grid }.clipped()` so it can't drive the foreground layout (and don't stack an opaque `Color` in front of a `.background` layer — it hides it).
- **Keep every toast's glass identical; status goes in the icon, not the surface**: tinting the glass per status makes one toast read as a near-solid fill while neutral ones refract — inconsistent. Use neutral `.regular` glass for all toast surfaces (capsule + card) and carry status color in the SF Symbol / tinted icon well. The opposite of the tinted-glass-for-destructive-buttons rule (toasts want consistency, controls want signalling).
- **Toast position respects the safe area for free**: `.overlay(alignment: .top)` aligns to the safe-area top, so `.padding(.top, 10)` sits below the Dynamic Island — a fixed pad lands under the notch when the view ignores the top safe area.
- **Auto-dismiss with a token** (`if toast?.id == new.id`) so a newer toast isn't cut short; slide via `.transition(.move(edge: .top).combined(with: .opacity))`.
- **Icon-only Liquid Glass action button**: `.buttonStyle(.glass)` + `.buttonBorderShape(.circle)` (material-circle fallback < iOS 26), cycling actions with `.contentTransition(.symbolEffect(.replace))` driven inside `withAnimation`; status haptic ladder (`notificationSuccess/Warning/Error`, `lightImpact` for neutral).
- Version output bumped to `⚙️ swiftui-microinteractions v1.12.0`

---

## [1.11.0] — 2026-06-20

Learnings from building UniqueLoaders (Canvas loaders that trace a shape's outline with a fading comet trail; swapped traced brand-logo shapes for generic geometric ones without touching motion).

- **Canvas Path Loaders section (new)**: drive a `Canvas` from a `TimelineView` clock and *precomputed* outline samples — never re-evaluate the shape math every frame. Captures the architecture and traps: **sample-then-animate** (cache `[CGPoint]` unit samples; animate only a `progress` head that interpolates along them, so the renderer is shape-agnostic and adding a shape needs zero motion-code changes); **arc-length-even sampling = constant-velocity motion** (equal-`t` sampling bunches points and makes the head speed up/slow down at corners — equal arc-length glides uniformly; shape sharpness and motion smoothness are independent); **two samplers** (polyline arc-length for crisp vertices vs closed Catmull-Rom for organic curves, which needs `>3` control points); shape recipes for star (`2N` vertices alternating outer `1.0`/inner `~0.42`), regular polygons, and orientation offsets.
- **Loader = clock, not a transition**: use `TimelineView(.animation)` deriving `progress = (time/duration).wrappedUnit` (+ independent rotation/breathe from the same clock), never a spring/`withAnimation` — the Canvas form of the existing "Loading Indicator → never spring" rule.
- **Content taste**: prefer generic geometric primitives (star/pentagon/hexagon/infinity) over tracing brand/trademark logos in a generic showcase — same premium feel, no trademark exposure, better generalization.
- Version output bumped to `⚙️ swiftui-microinteractions v1.11.0`

---

## [1.10.0] — 2026-06-19

Learnings from building MultiActionFABView (a speed-dial FAB whose actions extrude out of the button as liquid metaballs, in plain SwiftUI).

- **Canvas Metaball section (new)**: the `Canvas` goo recipe — `addFilter(.alphaThreshold)` + `addFilter(.blur)` over filled circles — for fuse/split liquid without iOS 26 `GlassEffectContainer`. Captures the traps: `alphaThreshold` yields a HARD edge (crisp at rest, gooey only on overlap); animate the Canvas by making the layer `View, Animatable` (`animatableData = progress`) or it jumps; keep blobs ~constant size so one global blur stays proportional (a shrinking bubble collapses the neck into a line); rest spacing must exceed `diameter + 2·blur` or blobs stay teardrop-fused; slow the spring (`response ≈ 1.0`) so the goo reads.
- **Canvas-vs-interaction split**: the Canvas is visual only — real `Button`s sit on top at matching positions; icons must travel with their blob on the **same spring** (animated `.position`), not `.transition`-pop at the slot; closed buttons need `.allowsHitTesting(false)` so they don't steal the FAB tap.
- **FAB styling taste**: a liquid FAB reads best **flat** (no shadow); blob + icon colors should be parameters that invert with the background (dark bg → white FAB/dark glyphs, light bg → dark FAB/white glyphs); `+`→`✕` is a 135° rotation of one `plus`; `.thin` weight for a premium glyph.
- **Themeable demo components**: make a view generic over its `Background` with a `Color`-default convenience init so existing call sites keep working while previews inject a `List`.
- Version output bumped to `⚙️ swiftui-microinteractions v1.10.0`

---

## [1.9.0] — 2026-06-14

Learnings from building GlassNotificationStackView (an Apple-style glass notification stack on an "Enable Push Notifications" screen).

- **Stacked Cards (notification stack)**: the collapsed peek-stack pattern — front card full-size, older cards offset/scaled/dimmed by depth; tap-to-expand into a list; swipe-the-front-card-to-dismiss (gated to `depth == 0`); cap the count so the oldest drops off the back. Includes the per-depth transform math and the insert-front-then-cap spring.
- **Custom Transitions — rubber-band entrance (`widthPop`)**: `.transition(.scale)` only scales *to* 1.0, so it can't enter oversized. Documented the `.modifier(active:identity:)` recipe with an **anisotropic** `scaleEffect(x:1.16, y:1.07)` (reads as a width stretch) + offset + opacity, and the key insight that the **rubber-band/overshoot comes from the spring's `dampingFraction` (~0.55), not the transition**.
- **Adaptive light + dark** (extends the Light Theme section): drive off `@Environment(\.colorScheme)` + semantic `.primary`/`.secondary` text; never hardcode `.white`/`.black` or force `.preferredColorScheme`; scheme-dependent values must be computed `var`s, not stored `let`s.
- **Dark glass cards — use the official Liquid Glass API**: on iOS 26 prefer `.glassEffect(.regular.tint((isDark ? .black : .white).opacity(0.28)), in: shape)` over faking it with material — real specular/refraction, tinted dark. Gate with `if #available(iOS 26.0, *)`; fall back to `.ultraThinMaterial` + scheme-aware tint on iOS 18–25. (An untinted `glassEffect` reads light — always tint for dark surfaces.)
- Version output bumped to `⚙️ swiftui-microinteractions v1.9.0`

---

## [1.8.0] — 2026-06-14

Learnings from building PowerAnalyticsCardView (light-theme dashboard in a bottom sheet) and the carousel mechanics behind ApplePayCardCarouselView.

- **`PressableScale` press-feedback primitive**: a reusable `ViewModifier` giving every row/button a real tactile press. Key rules encoded — use `DragGesture(minimumDistance: 0)` not `ButtonStyle` (the latter can't fire a haptic on the press-*down* edge); `lightImpact` on the down edge only; **lift-inside-to-fire** (drag off ~20pt cancels); attach via `.simultaneousGesture` so it doesn't block parent scroll; `0.90` scale for round buttons, `0.965` for rows.
- **Entrance / Appear Animation pattern**: staggered `opacity + offset(y:) + scaleEffect(anchor:.top)` gated on one `@State` flag with `.delay(index * 0.07)`, flipped in `.onAppear`. Documented the nuance that `@State` resets on each `.sheet` presentation, so the cascade re-fires for free every open — plus the don't-init-to-destination verification trap.
- **Light Theme section** (the skill was dark-only): a tonal token table (screen `0.92` / surface white / well `0.95` / track `0.88` / ink `0.10` / muted `0.55`) and the rule that **depth comes from tonal deltas, not shadows** — no drop shadows between stacked light surfaces; deepen accent colors for contrast on white; dark capsule CTA as the light-theme glass-button equivalent.
- **Data Dashboard Patterns**: animated proportional capsule bar (GeometryReader fill growing from 0 on appear, re-springing on data change) and `.numericText` used as a count-up for a hero metric driven by a segmented selector.
- **Carousels & Paging** (from ApplePayCardCarouselView): velocity-aware paging via `predictedEndTranslation` (flick detection) rather than raw translation; fan/card-stack layout (offset/scale/zIndex by distance to selected index); `* 0.5` drag-resistance multiplier; `selectionChanged` per card switch; MeshGradient 9-point premium card fill.
- Version output bumped to `⚙️ swiftui-microinteractions v1.8.0`

---

## [1.7.0] — 2026-06-13

Learnings from building ApplePayCardCarouselView — MeshGradient cards, sheet wrapping, and numericText transition.

- **SourceKit `HapticFeedback` false positive**: SourceKit reports `Cannot find 'HapticFeedback' in scope` in files written to `Carousels/` or `Animations/`. It is a single-file analysis artifact — `HapticFeedback.swift` is in the module Sources and resolves at compile time. Rule: never alter the code in response to this diagnostic.
- **Sheet wrapping pattern**: when a demo view presents content as a native `.sheet`, use two structs in the same file — public outer launcher (trigger button + `.sheet(isPresented:)`) and private inner sheet content (with `@Environment(\.dismiss)`). `private` model types in the same file are visible to both structs without access-level changes.
- **Numeric text transition — split currency prefix**: `.contentTransition(.numericText(value:))` on `"$173.11"` glitches the `$` during the digit roll. Split into a static `Text("$")` and an animated `Text(String(format: "%.2f", amount))`. Use `selectionChanged` haptic (not `mediumImpact`) when the transition is driven by a discrete scrub step.
- Version output bumped to `⚙️ swiftui-microinteractions v1.7.0`

---

## [1.6.0] — 2026-06-12

Learnings from analyzing over-specified prompts — why users had to write technical details that the skill should infer automatically.

- **Intent Inference step**: new pre-generation pass that resolves under-specified prompts to smart defaults and prints a `🔍 Inferred from prompt:` block. Users can see and override every auto-resolved detail before code is written.
- **Symbol → Effect lookup table**: stroke-based symbols (`signature`, `pencil`, `checkmark`, `wifi`…) auto-get `.drawOn` auto-loop; flood-fill shapes (`heart.fill`, `star.fill`) auto-get `.bounce value:` since drawOn is invisible on them; animated-structure symbols get `.variableColor.iterative.reversing` or `.breathe`; rotation symbols get `.rotate`.
- **Showcase mode vs. interaction mode auto-detection**: if the prompt contains no user action ("tap to", "on submit", "when user"…) → showcase mode with `.task` auto-loop. Only gates behind a tap when the prompt explicitly describes one. Eliminates the most common "nothing animates" failure.
- **Defaults Contract**: explicit rule that any UI element absent from the prompt defaults to the Apple HIG primitive — `.sheet(isPresented:)` for containers, `Button("Done")` / `Button("Clear")` in `.toolbar`, `NavigationStack` for multi-step flows. Never invents a custom modal when a system one fits.
- **Drawing/signature context rule**: prompts containing "signature", "draw", "sketch", or "handwrite" automatically get `.sheet`, toolbar Clear + Done, no custom dark background — without the user needing to spell any of it out.
- **Cross-reference in SF Symbols section**: added a callout pointing readers to the Intent Inference table for auto-effect selection, so the lookup is always found from both directions.
- **Archetype Catalog**: 8 named archetypes (Symbol Showcase, Liquid Toggle, Gesture Card, Sheet Interaction, Data Dashboard, Glass Morph, Particle/Physics Sim, Loading Indicator) each with their canonical container, physics preset, and haptic defaults. The `🎯 Archetype:` output line is now meaningful — it drives the rest of the generation, not just a label.
- **Haptics context rules**: explicit add/skip decision tree. Skip for pure symbol showcases, loaders, and ambient animations. Add for threshold gestures, toggles, destructive actions, and scrub steps. Includes a 4-rung haptic ladder (light → medium → heavy → selectionChanged) tied to gesture arc phases.
- **Physics ↔ Archetype mapping**: each archetype has a default physics preset so physics choices are deterministic when the prompt doesn't name a feel. Notably: Symbol Showcase → `.easeInOut` (symbol effects own timing), Loading Indicator → `.linear` (springs look jittery in loops), Glass Morph → 3-phase rubber-band.
- **Designer Vocabulary Map (Step 0 of Intent Inference)**: normalization pass that translates designer words into skill terms before any other inference runs. Covers three domains: animation feel words → physics presets ("snappy" → UI pop, "bouncy" → Snap, "stretchy" → rubber-band, "melts" → Slow morph), motion principle words → archetype signals ("flood" → Liquid Toggle, "anticipation" → pre-stretch, "wiggle" → `.wiggle`, "ripple" → radial Circle scale), and UI element names → SwiftUI primitives (13 aliases: "tray/drawer/panel" → `.sheet`, "toast/snackbar/nudge" → custom overlay, "skeleton/shimmer" → `redacted`, "FAB" → absolute ZStack button, etc.). Non-technical prompts now produce the same output as fully specified ones.
- Version output bumped to `⚙️ swiftui-microinteractions v1.6.0`

---

## [1.5.0] — 2026-06-11

Learnings from building SignatureSheetView — a `.drawOn` signature inside an Apple default bottom sheet. Root-caused why the first build's draw animation looked "broken."

- **`.drawOn` only plays on a *transition*, never on appear**: `isActive:` traces the path when the bound value *changes* while the view is visible — it does NOT replay when the view appears already in the visible state. A correct, present `.drawOn` modifier can still look broken if nothing ever fires the transition.
- **"draw a demo" / "animate X" → auto-play by default**: when the prompt asks the symbol to draw/animate itself (showcase, hint, loading), drive it with the `.task` loop. Only gate behind a tap/state change when the prompt explicitly names a user action ("tap to sign", "on submit"). Reaching for an interaction trigger when the user wanted a self-playing demo is the #1 way a drawOn build appears non-functional.
- **Verification trap — never validate a transition by pre-setting state to its destination**: initializing `@State` to the end value (e.g. `isHidden = false`) renders the symbol *already fully drawn, with no transition*, so a screenshot falsely confirms "it works." That validates layout, not motion. Verify by launching with the auto-loop (or scripting the tap) and capturing a **mid-trace frame**. Applies to every `isActive:`/`value:` symbolEffect.
- Version output bumped to `⚙️ swiftui-microinteractions v1.5.0`

---

## [1.4.0] — 2026-06-11

Learnings from debugging SF Symbols 7 `.drawOn` for a Medium article demo.

- **`.drawOn` isActive semantics are inverted**: `isActive: true` = hidden (pre-draw state), `isActive: false` = visible (animation plays on that transition). Toggling back to `true` plays draw-off automatically. This is the single most common `.drawOn` bug.
- **Symbol choice for `.drawOn`**: flood-fill symbols (`heart.fill`, `star.fill`) have no stroke path — drawOn jumps from hidden to fully filled with no visible trace. Use stroke-based symbols (`checkmark`, `heart` outline, `signature`, `wifi`) where the pen trace is clearly visible.
- **Never stack `.drawOn` + `.drawOff` on the same image**: both modifiers conflict and keep the symbol invisible.
- **Other SF Symbol effects documented** (iOS 17+): `.bounce` / `.wiggle` (discrete, value-triggered), `.breathe` / `.variableColor` / `.rotate` (indefinite, isActive), `.contentTransition(.symbolEffect(.replace))` (symbol swap — requires `withAnimation`).
- **`.task` over `Timer` for animation loops**: `.task` cancels automatically when the view disappears; `Timer` leaks. Pattern: `while !Task.isCancelled { try? await Task.sleep(for:); state.toggle() }`.
- Version output bumped to `⚙️ swiftui-microinteractions v1.4.0`

---

## [1.3.0] — 2026-05-30

Learnings from building GlassMorphActionView — an iOS 26 capsule ↔ red circle morphing toggle.

- **`GlassEffectContainer` morphing clusters**: documented pattern for fusing/separating multiple glass elements with metaball physics; uses `@Namespace` + `.glassEffectID()` for identity across state changes
- **Fusion-threshold spacing trap**: `GlassEffectContainer(spacing:)` is the fusion threshold — layout spacing between elements MUST exceed it or shapes blob together permanently at rest. Rule: `layoutSpacing > containerSpacing + 6pt`
- **Tinted glass**: `.glassEffect(.regular.tint(color), in: shape)` for colored states — never layer a colored shape on top of glass
- **Edge distortion trap**: never apply `.scaleEffect` with `.leading`/`.trailing` anchors to individual glass elements (subpixel artifacts distort edges even at scale 1.0). Apply container-wide stretch with `.center` anchor instead
- **3-phase rubber-band toggle**: pre-stretch (0.28/0.55) → morph (0.55/0.72) → release with overshoot (0.45/0.55) — stacked with `asyncAfter` for tactile rubber feel
- **Light gradient backgrounds for glass demos**: dark backgrounds hide refraction — use `LinearGradient([white:0.97, white:0.88])` to showcase iOS 26 glass
- **No redundant `.clipShape`** on top of `.glassEffect(in:)` — glassEffect already clips

---

## [1.2.0] — 2026-05-27

- **SF Symbol replace transition**: `.contentTransition(.symbolEffect(.replace.downUp))` pattern for toggle buttons (more ↔ close, play ↔ pause). Button stays visible — icon morphs in place. Never hide/show a separate close button.
- Version output bumped to `⚙️ swiftui-microinteractions v1.2.0`

---

## [1.1.0] — 2026-05-26

Improvements from real-world LiquidGlassTabBar development.

- **iOS 26 Liquid Glass**: `glassShape(radius:)` `@ViewBuilder` helper — `.glassEffect(in:)` on iOS 26+, `.ultraThinMaterial` fallback on older OS
- **ZStack overflow trap**: documented rule — conditionally render tall children + add `.clipped()`; `.opacity(0)` alone does NOT prevent frame overflow
- **Sliding tab indicator**: `GeometryReader` + `offset(x: tabW * selectedIndex)` pattern with exact fill/padding/radius spec matching iOS 26 HIG
- **Active indicator fidelity**: `Color.white.opacity(0.14)` solid fill, 5pt inset, corner radius = outerRadius − 5 — matches Apple Music / iOS 26 reference
- **Multi-component bar layout**: `HStack` of independent pill + circle components, not one monolithic ZStack
- **ContentView date**: use today's actual date in snippet, not hardcoded past date
- **Version line**: bumped to `⚙️ swiftui-microinteractions v1.1.0`

---

## [1.0.0] — 2026-05-21

Initial public release.

- Create mode: generates complete SwiftUI animation files from plain-English prompts
- Edit mode: reads existing file, applies targeted change, rewrites in place
- Asset scanning: detects `.imageset` folders in `Assets.xcassets` before generating
- Haptics check: uses shared `HapticFeedback.swift` if present, falls back to inline UIKit calls
- Xcode registration: auto-adds generated file to `.pbxproj` (PBXFileReference, PBXBuildFile, group child, Sources phase)
- Spring preset table: 7 presets mapped from feel words to exact `response`/`dampingFraction` values
- Progress output: streams `⚙️ v1.0.0` + 6 status lines so long generations feel responsive
- Supports 3 prompt levels: plain English · English + tech hints · fully technical
