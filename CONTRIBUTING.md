# CONTRIBUTING.md

## Agent Product Intentions & Decision Matrix

> **Target Audience**: AI Agents & Core Developers.  
> This document is the **product compass** for ytify. While [AGENTS.md](AGENTS.md) establishes _how_ to write code, this document governs _why_ ytify exists, _who_ we build for, _what_ we reject, and _how_ agents must evaluate product trade-offs.

---

## 📑 Index & Table of Contents

- [The Inception & Anti-Bloat Manifesto](#1-the-inception--anti-bloat-manifesto) - Why ytify was created in 2022, the NewPipe network crisis, and the ethical streaming philosophy.
- [The 4 Target Personas & User Empathy](#2-the-4-target-personas--user-empathy) - Empathy models for Constrained Network, Low-Spec Device, Mindful Listener, and Full-Agency users.
- [Product Anti-Goals & Taboos](#3-product-anti-goals--taboos) - Features, libraries, and patterns that agents must **never** propose or introduce.
- [Agent Decision Matrix & Evaluation Rubric](#4-agent-decision-matrix--evaluation-rubric) - The 4-step rubric every agent must run before proposing or writing code.
- [Technical Handoff to AGENTS.md](#5-technical-handoff-to-agentsmd) - Pointer to [AGENTS.md](AGENTS.md) for TypeScript, SolidJS, CSS, and layout standards.
- [Historical Release Evolution](#6-historical-release-evolution) - Milestone evolution history from v1 prototype to the modern dual-panel engine.

---

## 1. The Inception & Anti-Bloat Manifesto

### The Origin Story

In 2021, listening to background audio on mobile open-source clients (specifically NewPipe) on unstable or throttled mobile connections was an exercise in constant frustration. Audio playback stalled, stream resolution failed silently, and recovery required force-closing apps. When support issues languished, **ytify was founded in January 2022** on a single foundational thesis:

> **Audio streaming should be built for you, not against you.**

### Modern Web Streaming is Broken

Conventional commercial streaming applications (Spotify, YouTube Music) and even heavy web wrappers suffer from deep structural hostility toward users:

1. **Bloated Bundles**: Multi-megabyte JavaScript bundles, heavy telemetry, and third-party trackers consuming user bandwidth and battery.
2. **Attention Traps**: Infinite recommendation rabbit holes, algorithmic autoplay hooks, and engagement-maximizing discovery homepages designed to hijack attention.
3. **Network Fragility**: Heavy initial payloads that freeze or crash on high-latency, packet-loss-prone mobile connections.

### The ytify Stance

ytify is an intentional, ultra-lightweight web streaming client powered by YouTube, Piped, and Invidious backends. It treats user attention, mobile data, device battery, and memory as **scarce, precious resources**.

---

## 2. The 4 Target Personas & User Empathy

Every architectural decision and UI behavior must be evaluated through the lens of our four primary user profiles:

### 1. The Constrained Network User

- **Profile**: Operating on 2G, 3G, throttled 4G, or congested public Wi-Fi networks in emerging regions.
- **Pain Points**: High latency, dropped packets, aggressive data caps, expensive cellular bandwidth.
- **Agent Guardrails**:
  - Prioritize audio-only streams; defer or suppress video streams unless explicitly toggled.
  - Keep initial HTML/CSS/JS payload sizes as close to zero as possible.
  - Implement defensive stream resolution fallbacks and automatic reconnect retry policies.
  - Use lazy loading and thumbnail compression for media artwork.

### 2. The Low-Spec Device User

- **Profile**: Running budget Android smartphones, older laptops, or memory-limited embedded browsers.
- **Pain Points**: Low RAM (1–2 GB), thermal throttling, browser tab crashes caused by heavy Virtual DOM reconciliation.
- **Agent Guardrails**:
  - Leverage SolidJS fine-grained reactivity: mount DOM nodes once; never recreate components unnecessarily.
  - Strictly prevent memory leaks in audio event listeners and store subscriptions.
  - Keep CSS simple and hardware-accelerated (`transform`, `opacity`), avoiding complex multi-layer filter chains on low-tier screens.

### 3. The Mindful / Intentional Listener

- **Profile**: Seeking a calm, distraction-free audio player free from surveillance capitalism and algorithmic addiction feeds.
- **Pain Points**: Notification spam, forced discovery pages, algorithmic manipulation of listening habits.
- **Agent Guardrails**:
  - The UI defaults to the Player and Library—never an unprompted algorithmic discovery feed.
  - Zero autoplay traps: never start unprompted playback without user consent.
  - All recommendations (e.g., "For You") **must be computed locally on-device** using client-side history and saved favorites.

### 4. The Full-Agency Power User

- **Profile**: Demands granular, transparent control over their media library, playback pipeline, and viewport.
- **Pain Points**: Opaque queues, locked-in playlists, rigid mobile-only or desktop-only UI designs.
- **Agent Guardrails**:
  - Provide full queue control: reordering, batch deletions, clear queue, and instant shuffle.
  - Support offline caching via Progressive Web App (PWA) standards and audio downloads.
  - Preserve responsive layout autonomy: configurable landscape panel split ratios (`1:1`, `2:5`, `1:2`).

---

## 3. Product Anti-Goals & Taboos

To maintain ytify's integrity, AI agents must **strictly refuse** to implement or propose any of the following anti-patterns:

| Anti-Goal                           | Why It Is Prohibited                                                                                                                           |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Remote Telemetry / Analytics**    | ytify has zero trackers. Never send search terms, playback history, or device info to any remote telemetry service.                            |
| **Algorithmic Engagement Traps**    | No endless recommendation feeds, popup promos, or attention-hijacking carousels.                                                               |
| **Heavy External NPM Packages**     | Never install heavy dependencies (e.g., `lodash`, `moment`, `axios`, generic UI component libraries). Write lean, native TypeScript utilities. |
| **Global Navigation Chrome**        | No persistent bottom tab bars or top navigation bars. Navigation lives contextually inside in-view headers and dual-panel responsive layouts.  |
| **Disruptive Full-Page Navigation** | Navigation must never reload the page, trigger layout thrashing, or disrupt active media playback. Sub-views mount inside the Library panel.   |
| **Untyped Code (`any`, `unknown`)** | Never introduce `any` or lean on `unknown`. Model all domain data explicitly.                                                                  |
| **Forced Line-Cramming**            | Never artificially compress multiple statements onto one line. Clean, readable formatting is strictly required.                                |

---

## 4. Agent Decision Matrix & Evaluation Rubric

Before proposing any code change, new feature, or architectural modification, run through this 4-step evaluation rubric:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Agent Evaluation Rubric                         │
├────────────────────────────────────────────────────────────────────────┤
│ 1. The Payload Test    ───> Does this add unnecessary bundle bytes     │
│                             or make heavy network assumptions?         │
│ 2. The Device Test     ───> Does this increase memory footprint,       │
│                             thrash the DOM, or break SolidJS signals?  │
│ 3. The Code Economy    ───> Does this introduce a premature abstraction│
│    Test                     or violate the Pragmatic DRY principle?    │
│ 4. The Integrity Test  ───> Does this preserve user privacy, concrete  │
│                             typing, and proper formatting?             │
└────────────────────────────────────────────────────────────────────────┘
```

1. **The Payload Test**:
   - _Can this be accomplished with existing browser APIs instead of adding an external package?_
   - _Does this increase bundle weight for users on constrained 2G/3G connections?_
2. **The Device Test**:
   - _Will this run smoothly on a budget device without triggering garbage collection jank?_
   - _Does this preserve fine-grained SolidJS reactivity (no props destructuring)?_
3. **The Code Economy Test**:
   - _Does this abstraction have at least 3 proven, identical use cases?_
   - _Is the shared helper smaller and simpler than the inline code it replaces? (If no, keep it localized)._
4. **The Integrity Test**:
   - _Are all types concrete and explicit (strictly zero `any`, avoiding `unknown`)?_
   - _Is the code properly formatted with clean line breaks instead of forced single-line cramming?_
   - _Does this keep user data strictly client-side on-device?_

---

## 5. Technical Handoff to AGENTS.md

While this document governs product philosophy and agent decision-making, **all code implementation standards, architecture diagrams, and syntax rules are defined in [AGENTS.md](AGENTS.md)**:

- **High-Performance TypeScript**: Concrete domain models, monomorphic object shapes, $O(1)$ lookups, and compiler inference.
- **Smart Expressions & Clean Formatting**: Guard clauses, modern operators (`??`, `?.`, `??=`), and legible line breaks.
- **SolidJS Reactivity**: Signal accessors `()`, props preservation, `batch()` updates, and reactive control flows (`<Show>`, `<For>`).
- **CSS Optimization**: Native CSS nesting (`&`), modern pseudo-classes (`:is()`, `:where()`, `:has()`), and Open Props design tokens.
- **Dual-Panel Architecture**: Screen split ratios, scroll-snapping, MiniPlayer intersection lifecycle, and sticky-bar headers.
- **Pre-Flight Verification Checklist**: The final checklist before submitting code.

---

## 6. Historical Release Evolution

For historical context on how ytify's architecture progressed:

- [Version One](https://deploy-preview-8--ytify.netlify.app/) — Initial audio streaming prototype.
- [Version Two](https://deploy-preview-20--ytify.netlify.app/) — Core playback stabilization.
- [Version Three](https://deploy-preview-32--ytify.netlify.app/) — Queue management & theming introduction.
- [Version Four](https://deploy-preview-51--ytify.netlify.app/) — PWA support & offline foundation.
- [Version Five](https://deploy-preview-60--ytify.netlify.app/) & [Five.final](https://deploy-preview-118--ytify.netlify.app/) — UI overhaul & lyrics sync.
- [Version Six.beta](https://deploy-preview-124--ytify.netlify.app/) & [Six.lite](https://lite--ytify.netlify.app) — Performance tuning & lightweight edition.
- [Version Seven](https://deploy-preview-187--ytify.netlify.app/) & [Seven.Eight](https://67c6f71989c173000858cc5d--ytify.netlify.app/) — Dual-panel layout architecture.
- [Version SevenXEight](https://68bd4765e217420008188710--ytify.netlify.app/) — Modernized responsive engine.
