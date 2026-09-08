# swiftui-microinteractions &nbsp; [Try on Device](https://testflight.apple.com/join/jZXzPNEf)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![skills.sh installs](https://skills.sh/b/iAmVishal16/swiftui-microinteractions)](https://www.skills.sh/iamvishal16/swiftui-microinteractions)
[![Skill Stars](https://img.shields.io/github/stars/iAmVishal16/swiftui-microinteractions?style=flat&label=skill+stars)](https://github.com/iAmVishal16/swiftui-microinteractions/stargazers)
[![legendary-Animo](https://img.shields.io/github/stars/iAmVishal16/legendary-Animo?style=flat&label=legendary-Animo+⭐)](https://github.com/iAmVishal16/legendary-Animo/stargazers)

Premium SwiftUI animation and interaction skills for AI coding agents — generate production-ready micro-interactions from plain English prompts.

Built from [legendary-Animo](https://github.com/iAmVishal16/legendary-Animo): 84 hand-crafted SwiftUI animation demos.

---

## swiftui-microinteractions

Generate premium SwiftUI animations in the legendary-Animo style — spring physics, CoreHaptics, glass morphism, and complete compilable files — from a plain English description.

Each prompt writes a complete `.swift` file directly to your project. Supports both create and edit modes.

---

## Try These Prompts

Copy any prompt below, paste it into your AI agent, and get a production-ready `.swift` file instantly.

### Liquid & Metaball

```
/swiftui-microinteractions iOS toggle but the track floods with liquid when switched
```

```
/swiftui-microinteractions a speed-dial FAB whose action buttons extrude as liquid metaballs
```

```
/swiftui-microinteractions a liquid step slider where fill and thumb are one animatable metaball
```

### Gesture-Driven Cards

```
/swiftui-microinteractions a notification card you can rip apart by pulling both edges
```

```
/swiftui-microinteractions a Tinder-style swipeable card deck with velocity-aware fling
```

```
/swiftui-microinteractions a stacked notification deck that fans open on tap with spring physics
```

### Glass & Morphing (iOS 26)

```
/swiftui-microinteractions a capsule that morphs into a red circle toggle with rubber-band physics
```

```
/swiftui-microinteractions a liquid glass tab bar with a sliding indicator that morphs between tabs
```

```
/swiftui-microinteractions glass toast banners that slide in from the top over a photo grid
```

### SF Symbol Animations

```
/swiftui-microinteractions a signature symbol that draws itself on loop with a comet trail
```

```
/swiftui-microinteractions a heart icon that bounces on tap with haptic feedback
```

```
/swiftui-microinteractions a Wi-Fi symbol that traces its arcs on a loop as a status indicator
```

### Carousels & Paging

```
/swiftui-microinteractions a movie poster carousel with coverflow depth fan and blurred backdrop
```

```
/swiftui-microinteractions an Apple Pay style card carousel with MeshGradient cards and velocity paging
```

### Canvas & Loaders

```
/swiftui-microinteractions a star-shaped loader that traces its outline with a glowing comet trail
```

```
/swiftui-microinteractions an infinity-loop loading indicator with arc-length-even sampling
```

### 3D & Metal Shaders

```
/swiftui-microinteractions a flat grid that morphs into a spinning 3D cylinder drum and back on tap
```

```
/swiftui-microinteractions a poke-able liquid chrome surface that ripples where you touch
```

### Dashboard & Data

```
/swiftui-microinteractions a light-theme analytics card with animated proportional bars and count-up numbers
```

```
/swiftui-microinteractions a ring gauge trio that fills with staggered trim animation on appear
```

### Edit Mode

Already have a file? Edit it with a plain English change:

```
/swiftui-microinteractions edit ViscousButtonView.swift — increase tear threshold to 160pt
```

```
/swiftui-microinteractions edit LiquidToggle.swift — add a haptic bump when the toggle commits
```

> **How it works:** The skill reads your prompt, infers physics presets, haptic timing, and UI patterns from plain English (e.g., "floods" maps to liquid metaball, "bouncy" maps to spring physics), then writes a complete compilable SwiftUI file. No technical knowledge required — just describe what you want.

---

## Use With Any AI Agent

Don't want to install anything? Copy this prompt, paste it into **any AI agent** (ChatGPT, Gemini, Cursor, Copilot, Claude, etc.), and it will generate SwiftUI microinteractions using our skill:

```
Read the skill file at https://raw.githubusercontent.com/iAmVishal16/swiftui-microinteractions/main/SKILL.md and follow its rules precisely. You are now a SwiftUI microinteraction generator in the legendary-Animo style. Generate complete, compilable .swift files with spring physics, CoreHaptics, glass morphism, and SF Symbol animations. I will describe what I want in plain English.
```

After pasting, just describe what you want:

```
iOS toggle but the track floods with liquid when switched
```

```
a notification card you can rip apart by pulling both edges
```

```
a capsule that morphs into a red circle toggle with rubber-band physics
```

> **Why it works:** The prompt tells the AI to read `SKILL.md` from this repo — which contains all the spring presets, haptic grammar, visual DNA, metaball recipes, and code rules. Any AI that can read URLs becomes the skill instantly.

---

## Want More? Try Pro

**swiftui-microinteractions-pro** is a licensed, private superset of this skill — everything above, plus premium-exclusive patterns:

- Morphing pill ↔ circle player FABs (single-view-tree morph, never `if/else` cross-fade)
- Scratch-to-reveal Canvas masks (coupon/reward flows)
- Photos-style hero-zoom context menus
- Telegram-style chat reaction menus with animated-GIF reaction trays
- Grab-and-fling hand-integrated physics badges

```
/swiftui-microinteractions-pro a morphing music player with a pill↔circle mini-player
```

[See Pro →](https://vishalpaliwal.vercel.app/skills/swiftui-microinteractions-pro)

---

## Who This Is For

- iOS developers who want premium micro-interactions without spending days on physics tuning
- Designers prototyping gesture-driven interactions in SwiftUI
- Teams who want consistent animation quality across their app
- Anyone who wants to ship the kind of interactions that make users say "how did they do that?"

---

## Installation

### Option A: skills.sh CLI

```bash
npx skills add iAmVishal16/swiftui-microinteractions
```

Then use in your agent:
```
/swiftui-microinteractions iOS toggle but the track floods with liquid when switched
```

[View on skills.sh →](https://www.skills.sh/iamvishal16/swiftui-microinteractions)

### Option B: Claude Code Plugin

**Add to your `.claude/settings.json`:**

```json
{
  "enabledPlugins": {
    "swiftui-microinteractions@iamvishal16-skills": true
  },
  "extraKnownMarketplaces": {
    "iamvishal16-skills": {
      "source": {
        "source": "github",
        "repo": "iAmVishal16/swiftui-microinteractions"
      }
    }
  }
}
```

**Or via Claude Code CLI:**
```
/plugin marketplace add iAmVishal16/swiftui-microinteractions
/plugin install swiftui-microinteractions@iamvishal16-skills
```

### Option C: Manual Install

```bash
curl -o ~/.claude/commands/swiftui-microinteractions.md \
  https://raw.githubusercontent.com/iAmVishal16/swiftui-microinteractions/main/SKILL.md
```

Skill is then available as `/swiftui-microinteractions` in any Claude Code session.

---

## What's Inside

The skill encodes the full legendary-Animo aesthetic without requiring knowledge of the codebase:

- **Spring physics library** — 7 tuned presets (snap, pop, settle, morph, stiff, dial) with exact `response` + `dampingFraction` values
- **Haptic grammar** — 4-event ladder tied to interaction phases (drag start, threshold cross, commit, destroy)
- **Visual DNA** — dark background, glass morphism surfaces, 7-level opacity hierarchy, two-tone gradient system
- **Liquid metaball pattern** — `Canvas` + `.blur()` + `.contrast()` + `.blendMode(.screen)` recipe
- **Multi-phase animation chains** — stacked `DispatchQueue.main.asyncAfter` for choreographed sequences
- **State architecture tiers** — Simple (2–4 `@State`) / Medium (5–8) / Complex (10+) with property type rules
- **Code structure law** — mandatory `MARK` layout, camelCase tokens, no magic numbers
- **Create + Edit modes** — generates new files or modifies existing ones, writes directly to disk

---

## Skill Structure

```
swiftui-microinteractions/   ← repo root
  SKILL.md
  README.md
  CHANGELOG.md
  LICENSE
```

Single-skill repo: `SKILL.md` lives at the repo root, indexed by skills.sh as `iamvishal16/swiftui-microinteractions`.

---

## Contributing

Contributions welcome. Open a PR to improve the skill content, add new animation patterns, or fix incorrect physics values.

---

## About

Built by [Vishal Paliwal](https://twitter.com/iamvishal16_ios) — iOS developer and creator of [legendary-Animo](https://github.com/iAmVishal16/legendary-Animo).

Support the work: [Patreon](https://www.patreon.com/c/iamvishal16)

---

## License

MIT License. See [LICENSE](LICENSE) for details.
