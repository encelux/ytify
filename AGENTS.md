# AGENTS.md

## Application Architecture & Engineering Standards

> Comprehensive architectural reference, performance principles, and coding standards for **ytify**. All AI agents and human contributors must strictly adhere to these guidelines. For the overarching project philosophy, user personas, and contributor guidelines, refer to [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📑 Index & Table of Contents

- [Core Philosophy & Guiding Tenets](#1-core-philosophy--guiding-tenets) - "Less code is better", proper formatting over forced compression, clean code, fine-grained reactivity, and [CONTRIBUTING.md](CONTRIBUTING.md) link.
- [Pragmatic DRY & Code Economy](#2-pragmatic-dry--code-economy) - Why duplication is cheaper than the wrong abstraction, avoiding forced DRY when wrappers exceed inline code, and code economy.
- [High-Performance Modern TypeScript](#3-high-performance-modern-typescript) - High-performance TS/JS, strict prohibition of `any`, avoiding `unknown`, engine-friendly patterns, zero-cost type safety, and GC reduction.
- [Smart Expressions & Proper Formatting](#4-smart-expressions--proper-formatting) - Eliminating boilerplate, guard clauses, optional chaining, nullish coalescing, readable line breaks, and avoiding line-cramming.
- [SolidJS Fine-Grained Reactivity Performance](#5-solidjs-fine-grained-reactivity-performance) - Signal accessor rules, avoiding prop destructuring, purposeful memoization, reactive batching (`batch()`), and native DOM control flows.
- [High-Efficiency CSS for Code Size](#6-high-efficiency-css-for-code-size) - Minimizing stylesheet weight via native CSS nesting (`&`), modern pseudo-classes (`:is`, `:where`, `:has`), modern shorthands, and token reuse.
- [Dual-Panel Screen Architecture & Responsive Layout](#7-dual-panel-screen-architecture--responsive-layout) - Player (left) and Library (right) panel architecture, Portrait scroll snapping + MiniPlayer lifecycle, and Landscape split ratios.
- [Navigation & History Contract](#8-navigation--history-contract) - Sub-views (Search, Settings, List), `history.pushState` integration, and non-disruptive browser back button handling.
- [Unified Header Architecture (`header.sticky-bar`)](#9-unified-header-architecture-headersticky-bar) - Canonical sticky-bar header pattern, title truncation, `.right-group` action alignment, and `<details>` dropdowns.
- [State Management & Circular Dependency Prevention](#10-state-management--circular-dependency-prevention) - Store responsibilities (`player`, `queue`, `navigation`, `app`), decoupled configuration readers, and architectural firewalls.
- [Asset & Icon Optimization](#11-asset--icon-optimization) - Custom RemixIcon glyph subset, zero unused icon classes, and lightweight web asset consumption.
- [Code Health & Review Checklist](#12-code-health--review-checklist) - Pre-flight verification checklist for agents and contributors before proposing changes.

---

## 1. Core Philosophy & Guiding Tenets

**ytify** is a lightweight, high-performance, client-side web application for streaming music and videos powered by YouTube, Piped, and Invidious sources. Built with **SolidJS** and **TypeScript**, it prioritizes near-instant startup, zero superfluous re-renders, tiny bundle size, and adaptive responsiveness across mobile and desktop displays.

> [!NOTE]
> **Project Manifesto & User Personas**: For the story behind ytify's inception, our 4 primary user archetypes (constrained networks, low-spec devices, mindful listeners, power users), and product intentions, see [CONTRIBUTING.md](CONTRIBUTING.md).

### Primary Engineering Pillars

1. **Less Code is Better**: Every line of code is a liability, a potential bug vector, and extra maintenance overhead. Write only what solves the problem directly. Do not build speculative scaffolding or premature abstractions for hypothetical future requirements.
2. **Proper Formatting Over Artificial Line Compression**: "Less code is better" means eliminating unnecessary abstractions, bloated wrappers, ceremonial boilerplate, and dead code. It does **not** mean artificially cramming multiple statements onto single lines or writing compressed, unreadable expressions. Source code must remain properly formatted, well-spaced, and clean. Automated minification and byte packing belong to the build tool (Vite / Rollup / esbuild), never the source author.
3. **Clean Code Over Cleverness**: Code should be immediately self-evident and readable. Avoid cryptic one-liners, overly dense regular expressions, or convoluted multi-layered indirection.
4. **Fine-Grained Reactivity**: Maximize SolidJS's fine-grained reactive model. Work directly with signal accessors without the virtual DOM overhead of React. Components mount once; only individual reactive DOM bindings update.
5. **Zero Global Navigation Chrome**: The app does not use persistent global tab bars or navigation footers. Navigation is contextual, living inside in-view headers and dual-panel responsive arrangements.

---

## 2. Pragmatic DRY & Code Economy

### The Pragmatic DRY Principle

**Forced DRY everywhere is strictly forbidden.** Premature abstraction is far more expensive than localized duplication. Duplication is far cheaper than the wrong abstraction.

- **The Threshold Rule**: Never introduce a shared abstraction, generic helper, or custom wrapper component for fewer than three distinct, proven use cases that share identical business invariants.
- **The Code Weight Rule**: If a shared "DRY" helper requires more lines of code, additional configuration options, or higher conceptual overhead than the duplicated code it replaces, **keep the code inline and localized**.
- **Structural Similarity vs. Domain Coupling**: Just because two pieces of code look identical today does not mean they share the same domain reason to change. Decoupled, local code is simpler to refactor and delete.

```typescript
/* ❌ ANTI-PATTERN: Over-engineered DRY helper for two trivial operations */
interface ClickHandlerConfig<T> {
  action: (item: T) => void;
  stopPropagation?: boolean;
  preventDefault?: boolean;
  logEvent?: string;
}
function createGenericClickHandler<T>(config: ClickHandlerConfig<T>) {
  return (e: MouseEvent, item: T) => {
    if (config.stopPropagation) e.stopPropagation();
    if (config.preventDefault) e.preventDefault();
    if (config.logEvent) console.debug(config.logEvent, item);
    config.action(item);
  };
}

/* ✅ CODEBASE STANDARD: Clean, direct, self-contained inline expressions */
<button onclick={(e) => { e.stopPropagation(); playTrack(track); }}>Play</button>
<button onclick={(e) => { e.stopPropagation(); removeFromQueue(index); }}>Remove</button>
```

---

## 3. High-Performance Modern TypeScript

TypeScript in **ytify** must compile into lean, fast-path JavaScript with zero unnecessary runtime penalty and minimal Garbage Collection (GC) pressure.

### A. Lean Type Signatures & Leveraging Inference

- Rely on TypeScript's inference engine for local variables, constants, and straightforward return values. Redundant manual type annotations add visual noise and maintenance overhead.
- Use `import type` for type-only imports to ensure bundler treeshaking removes all traces at build time.
- Prefer `type` aliases and discriminated unions over class hierarchies or complex interface merging.

```typescript
/* ❌ ANTI-PATTERN: Redundant manual annotations and boilerplate interfaces */
const trackCount: number = 10;
const isPlaying: boolean = false;
const formatTitle = (raw: string): string => raw.trim();

interface TrackPayload {
  id: string;
  kind: "audio" | "video";
}
function handleTrack(track: TrackPayload) {
  /* ... */
}

/* ✅ CODEBASE STANDARD: Clean inference & zero-cost type constructs */
import type { TrackItem } from "@types";

const trackCount = 10;
const isPlaying = false;
const formatTitle = (raw: string) => raw.trim();

type TrackPayload = { id: string; kind: "audio" | "video" };
function handleTrack(track: TrackPayload) {
  /* ... */
}
```

### B. Execution Performance & Garbage Collection Optimization

- **Avoid multi-pass array allocations**: Do not chain `.filter().map().filter()` when handling sizable lists. Consolidate passes using a single loop, `.reduce()`, or an inline generator when processing larger datasets.
- **Fast lookups**: Use native `Set` or `Map` for $O(1)$ presence checks instead of repeated $O(N)$ searches via `array.includes()` or `array.some()`.
- **Monomorphic object shapes**: Keep object structures consistent. Avoid dynamically adding or deleting keys from hot objects, as this de-optimizes V8 hidden classes.

```typescript
/* ❌ ANTI-PATTERN: Multi-pass intermediate arrays on large collection */
const activeIds = items
  .filter((item) => item.isActive)
  .map((item) => item.id)
  .filter((id) => !blockedIds.includes(id));

/* ✅ CODEBASE STANDARD: Single-pass transformation with Set lookup */
const blockedSet = new Set(blockedIds);
const activeIds: string[] = [];
for (const item of items) {
  if (item.isActive && !blockedSet.has(item.id)) {
    activeIds.push(item.id);
  }
}
```

### C. Strict, Concrete Typing: Prohibition of `any` & Avoiding `unknown`

Untyped escape hatches are unacceptable in ytify:

- **`any` is strictly prohibited**: It circumvents compiler guarantees, hides defects, and degrades IDE intellisense.
- **`unknown` should be avoided**: It is frequently used as a lazy substitute for proper modeling, forcing repetitive runtime checking ceremonies and unsafe type assertions (`as Type`).
- **The Standard: Explicit, Well-Modeled Domain Types**:
  - Always model API responses, events, and payload shapes with **explicit domain types**.
  - Use **discriminated unions** to model multi-state responses or polymorphic objects.
  - Use **constrained generics** (`<T extends TrackItem>`) for reusable algorithms instead of vague generic parameters.
  - Define explicit dictionary signatures (`Record<string, string>`, `Record<ActionKey, () => void>`) rather than vague object maps.

```typescript
/* ❌ ANTI-PATTERN: Using 'any' or fallback 'unknown' escape hatches */
function parseTrack(payload: any) {
  return payload.title;
}

function handleResponse(data: unknown) {
  // Messy assertion boilerplate because data was not modeled
  return (data as { items: Array<{ title: string }> }).items;
}

/* ✅ CODEBASE STANDARD: Explicit, well-modeled domain types & discriminated unions */
type TrackPayload = {
  id: string;
  title: string;
  author: string;
  duration: number;
};

type StreamResult =
  | { status: "success"; track: TrackPayload }
  | { status: "error"; message: string };

function parseTrack(payload: TrackPayload): string {
  return payload.title;
}

function handleStreamResult(result: StreamResult): void {
  if (result.status === "error") {
    showToast(result.message);
    return;
  }
  playTrack(result.track);
}
```

---

## 4. Smart Expressions & Proper Formatting

Use concise, idiomatic modern JavaScript/TypeScript expressions to eliminate ceremony while maximizing readability. Formatting must remain clean and readable—never sacrifice readability by cramming statements onto one line.

### A. Guard Clauses Over Nested Conditionals

Flatten nested logic with early returns to eliminate pyramidal indentation and cognitive load.

```typescript
/* ❌ ANTI-PATTERN: Deep nested if-else structures */
function processStream(stream?: StreamItem) {
  if (stream) {
    if (stream.url) {
      if (!stream.isExpired) {
        return loadStream(stream.url);
      } else {
        return refreshStream(stream);
      }
    }
  }
  return null;
}

/* ✅ CODEBASE STANDARD: Guard clauses & early returns */
function processStream(stream?: StreamItem) {
  if (!stream?.url) return null;
  if (stream.isExpired) return refreshStream(stream);
  return loadStream(stream.url);
}
```

### B. Idiomatic Modern Operators

- **Nullish Coalescing (`??`)**: Use `??` when fallback values should apply only to `null` or `undefined` (preserving valid falsy values like `0` or `""`).
- **Optional Chaining (`?.`)**: Safely traverse nullable paths without defensive multi-line checks.
- **Logical Assignment (`??=`, `||=`, `&&=`)**: Update values in place cleanly without repetitive assignment statements.
- **Object Shorthand**: Always use `{ key }` over `{ key: key }`.

```typescript
/* ❌ ANTI-PATTERN: Verbose fallback and assignment boilerplate */
const vol =
  config.volume !== undefined && config.volume !== null ? config.volume : 100;
let cache = getCache();
if (!cache) {
  cache = initializeCache();
}

/* ✅ CODEBASE STANDARD: Smart modern expressions */
const vol = config.volume ?? 100;
cache ??= initializeCache();
```

### C. Clean Formatting vs. Forced Line-Cramming

Do not compress multiple distinct statements onto a single line to artificially reduce line count. Code must be cleanly formatted with standard line breaks and indentation.

```typescript
/* ❌ ANTI-PATTERN: Cramming multiple statements onto one line or unreadable nesting */
// Avoid collapsing discrete actions: if (isReady) { play(); notify(); setQueue(q); return true; }
const val = a ? (b ? c : d ? e : f) : g;

/* ✅ CODEBASE STANDARD: Clean, readable formatting with clear line breaks */
if (isReady) {
  play();
  notify();
  setQueue(q);
  return true;
}

// Flat, readable conditional branches over nested ternary ladders
if (!a) return g;
if (b) return c;
return d ? e : f;
```

---

## 5. SolidJS Fine-Grained Reactivity Performance

SolidJS is fundamentally distinct from virtual DOM frameworks (React, Vue). Components run **once** as setup functions; reactive updates occur exclusively through fine-grained signal subscriptions.

### A. Reactive Accessor Rules & Props Preservation

- **Never destructure props**: Destructuring component props immediately severs reactivity. Access props via `props.propertyName` or use `mergeProps`.
- **Signals are functions**: Always invoke signals with `()` inside JSX expressions and memos to subscribe to changes.

```typescript
/* ❌ ANTI-PATTERN: Destructuring props breaks SolidJS fine-grained reactivity */
export function MediaBadge({ title, duration }: { title: string; duration: number }) {
  // title and duration will never re-render when upstream signals change!
  return <span title={title}>{convertSStoHHMMSS(duration)}</span>;
}

/* ✅ CODEBASE STANDARD: Direct prop access preserves reactive subscriptions */
export function MediaBadge(props: { title: string; duration: number }) {
  return <span title={props.title}>{convertSStoHHMMSS(props.duration)}</span>;
}
```

### B. Prudent Memoization (`createMemo`)

- Only wrap computations in `createMemo` if they involve non-trivial algorithms, heavy array sorting, or large data transformations.
- Do not wrap cheap scalar operations (e.g., string concatenations, simple math, boolean checks) in `createMemo`. The memory and subscription overhead of the memo exceeds the cost of direct re-evaluation.

### C. Reactive Control Flow Elements

- Always use `<Show>`, `<For>`, `<Index>`, `<Switch>`, and `<Match>`.
- **Never** use raw `array.map()` inside JSX to render lists. `array.map()` rebuilds all DOM nodes upon any array mutation, completely bypassing Solid's surgical DOM reconciliation.
- **Batching Writes**: When an action triggers multiple signal or store mutations simultaneously, wrap them in `batch(() => { ... })` to suppress downstream recalculation churn until all mutations complete.

```typescript
/* ✅ CODEBASE STANDARD: Batching concurrent state updates */
import { batch } from "solid-js";

function selectAndPlay(track: TrackItem, index: number) {
  batch(() => {
    setPlayerStore("stream", track);
    setPlayerStore("playbackState", "loading");
    setQueueStore("currentIndex", index);
  });
}
```

---

## 6. High-Efficiency CSS for Code Size

Stylesheets in **ytify** must achieve maximum visual expressiveness with minimal byte size. Every selector, property, and rule must be lean and performant.

### A. Native CSS Nesting (`&` and Direct Combinators)

- Stylesheets must leverage native CSS nesting.
- Avoid flat, repetitive selector chains that inflate file size and increase parse times.
- Use direct child combinators (`>`) to bound styling scope without adding arbitrary class names.

```css
/* ❌ ANTI-PATTERN: Repetitive flat selectors inflating stylesheet size */
.player-queue-section {
  width: 100%;
}
.player-queue-section > header {
  display: flex;
  position: sticky;
}
.player-queue-section > header > p {
  flex: 1;
  text-overflow: ellipsis;
}
.player-queue-section > header .right-group {
  margin-left: auto;
  display: flex;
}
.player-queue-section > header details summary {
  cursor: pointer;
}

/* ✅ CODEBASE STANDARD: Compact native CSS nesting */
.player-queue-section {
  width: 100%;

  > header {
    display: flex;
    position: sticky;
    top: 0;
    z-index: 10;

    > p {
      flex: 1;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .right-group {
      margin-left: auto;
      display: flex;
      gap: var(--gap-sm);
    }

    details summary {
      cursor: pointer;
    }
  }
}
```

### B. Modern Pseudo-Classes (`:is()`, `:where()`, `:has()`)

- Use `:is()` and `:where()` to compress repetitive multi-target rules into single compact lines.
- Use `:has()` for parent and relational state styling, eliminating the need to toggle extra JavaScript state classes on parent containers.

```css
/* ❌ ANTI-PATTERN: Bloated multi-target rules */
.card-item:hover,
.card-item:focus-visible,
.stream-row:hover,
.stream-row:focus-visible {
  background-color: var(--onBg2);
}

/* ✅ CODEBASE STANDARD: Compressed rule with :is() */
:is(.card-item, .stream-row):is(:hover, :focus-visible) {
  background-color: var(--onBg2);
}

/* Relational styling via :has() without JS state classes */
section:has(> header > details[open]) > *:not(header) {
  filter: blur(var(--size-px-1));
  pointer-events: none;
}
```

### C. Modern Shorthands & Design Token Reuse

- Use logical properties and modern shorthands: `inset: 0`, `margin-inline`, `padding-block`, `gap`, `aspect-ratio`.
- Strictly reuse CSS variables from `src/styles/global.css` and Open Props (`var(--gap)`, `var(--gap-sm)`, `var(--roundness)`, `var(--bg)`, `var(--text)`, `var(--onBg)`). Never introduce hardcoded ad-hoc pixel spacers or arbitrary hex colors.
- **Consolidated Media Queries**: Group portrait and landscape overrides into unified blocks instead of scattering fragmented `@media` queries throughout individual selectors.

---

## 7. Dual-Panel Screen Architecture & Responsive Layout

The application is strictly partitioned into two primary functional screens/panels:

- **Left Panel (Player Panel)**: Houses playback controls, media details/artwork, lyrics, video renderer, and the integrated scrollable queue.
- **Right Panel (Library Panel)**: Houses user collections (playlists, history, saved streams) and acts as the mounting root for sub-views (`Search`, `Settings`, `List`).

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Dual-Panel Layout Model                         │
├───────────────────────────────────┬────────────────────────────────────┤
│            Left Panel             │            Right Panel             │
│          (Player Screen)          │          (Library Screen)          │
│ ───────────────────────────────── │ ────────────────────────────────── │
│  • Media Artwork & Details        │  • Default View: Library           │
│  • Playback Controls & Scrubber   │  • Sub-views: Search, List,        │
│  • Lyrics / Video Stream Target   │    Settings (pushed via history)   │
│  • Integrated Scrollable Queue    │  • Header: Search, Fullscreen,     │
│  • Sticky Queue Header & Tools    │    Settings controls               │
└───────────────────────────────────┴────────────────────────────────────┘
```

### A. Portrait Mode (Mobile / Narrow Viewport)

- **Horizontal Scroll Snapping**: Panels are positioned horizontally with native CSS scroll snapping (`scroll-snap-type: x mandatory`).
- **Touch Gesture Navigation**: Swiping horizontally navigates between the Player (left) and Library/sub-views (right).
- **MiniPlayer Lifecycle (`IntersectionObserver`)**:
  - When the **Right Panel** is visible and playback is active, the MiniPlayer mounts docked at the viewport bottom.
  - When the user scrolls into the **Left Panel**, the MiniPlayer unmounts automatically, revealing full player controls and queue.

### B. Landscape Mode (Desktop / Tablet)

- **Persistent Side-by-Side View**: Both panels render simultaneously.
- **MiniPlayer Suppression**: The MiniPlayer is strictly disabled and unmounted.
- **Configurable Width Split**:
  - Panel width split is user-configurable in Settings (`1:1`, `2:3`, `3:4`, `1:2`, `2:5`; default is `2:5`).
  - Applied persistently via dynamic CSS variables (`--panelRatioLeft`, `--panelRatioRight`).

---

## 8. Navigation & Sub-View History Contract

### Navigation Hierarchy

- **Root View**: `Library` is the persistent root view of the Right Panel.
- **Sub-Views**:
  - **Search**: Opened via the search button in the Library header.
  - **Settings**: Opened via the settings button in the Library header.
  - **List**: Displays playlists, albums, artist channels, and search results.

### History & Back Navigation Contract

- Opening any sub-view (Search, Settings, List) must perform a **history push** (`history.pushState` / router push).
- Triggering the browser/device **back button** (or header close/back action) pops the history state, closes the active sub-view, and restores the Library screen **without interrupting active media playback**.
- Never reload the page or reset the player store during navigation.

---

## 9. Unified Header Architecture (`header.sticky-bar`)

All feature views (`Library`, `List`, `Search`, and the integrated `Player Queue`) must implement the canonical header pattern defined in `src/styles/layout.css`:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        Canonical Header Pattern                        │
├────────────────────────────────────────────────────────────────────────┤
│ <header class="sticky-bar">                                            │
│   <p>Title / Counter</p>                 ───> flex-grow / text ellipsis│
│   <div class="right-group">...</div>     ───> margin-left: auto; flex  │
│   <Dropdown />                           ───> position: absolute;      │
│ </header>                                     top: var(--gap); right: 0│
└────────────────────────────────────────────────────────────────────────┘
```

### Header Design Rules

1. **Semantic Container**: Always use `<header class="sticky-bar">` inside feature sections.
2. **Title & Counter**: Placed directly as `> p` within `<header>` with single-line text ellipsis truncation (`overflow: hidden; text-overflow: ellipsis; white-space: nowrap;`).
3. **Action Button Groups**: Group secondary action icons inside `.right-group` (`margin-left: auto; display: flex; gap: var(--gap-sm);`).
4. **Dropdown & Tool Menus**: Menus render inside `<details>` / `<Dropdown />` anchored relative to the header.
5. **Glassmorphism & Backdrop**: Use the shared `.sticky-bar` background and `backdrop-filter: blur(...)` tokens rather than defining custom blur properties.

---

## 10. State Management & Circular Dependency Prevention

### Store Responsibility Map

- **`src/lib/stores/player.ts`**: Media playback lifecycle, HTML audio/video elements, playback rates, position/duration, volume, and visualizers.
- **`src/lib/stores/queue.ts`**: Active playback queue, play history, upcoming tracks, shuffle algorithms, and batch queue mutations.
- **`src/lib/stores/navigation.ts` & `src/lib/stores/app.ts`**: Coordinates active panel index, open sub-views, drawers, and responsive layout state.

### Circular Dependency Prevention

- Pure utilities (`src/lib/utils/config.ts`, `src/lib/utils/image.ts`) must never import reactive stores (`src/lib/stores/*`).
- Configuration readers, DOM property helpers, and persistent storage wrappers must remain strictly decoupled from store state to avoid runtime initialization errors (`ReferenceError: Cannot access before initialization`).

---

## 11. Asset & Icon Optimization

- **Custom-Subset RemixIcon**: The application bundles a custom glyph subset in `src/styles/remixicon.css` to minimize web font payload.
- Only utilize glyph classes included in the subset (e.g., `ri-play-fill`, `ri-pause-fill`, `ri-search-2-line`, `ri-close-large-line`, `ri-more-2-fill`, `ri-skip-back-fill`).
- Verify glyph class validity before referencing icons in JSX to prevent rendering blank glyph placeholders.

---

## 12. Code Health & Review Checklist

Before completing any task or opening a pull request, run through this verification checklist:

- [ ] **Strict Concrete Types (No `any`, Avoid `unknown`)**: Strictly zero `any` types and avoid `unknown`. All entities, API payloads, and parameters are modeled with explicit domain types, discriminated unions, or generics.
- [ ] **Proper Formatting**: Code is cleanly and legibly formatted with standard line breaks and indentation. No multiple statements forcefully crammed onto a single line.
- [ ] **Code Economy**: Is this implementation as minimal as possible? Are there any unnecessary abstraction layers, speculative functions, or unused files?
- [ ] **Pragmatic DRY**: If code was abstracted into a shared helper, are there at least 3 genuine use cases? Does the helper contain fewer lines of code and less complexity than the inline calls?
- [ ] **TypeScript Quality**: Are types strictly checked without superfluous manual annotations? Are type imports marked with `import type`?
- [ ] **Performance & Allocation**: Are there any unnecessary intermediate array copies, unmemoized expensive loops, or missed `Set`/`Map` lookup optimizations?
- [ ] **Reactivity Safety**: Are props passed without destructuring? Are signals called as accessors `()`? Are multiple mutations batched via `batch()`?
- [ ] **CSS Compactness**: Does the stylesheet use native nesting? Are design tokens (`var(--gap)`, `var(--onBg)`, etc.) used instead of arbitrary pixels or colors? Are pseudo-classes (`:is()`, `:has()`) utilized to avoid selector duplication?
- [ ] **Navigation & Layout**: Does the code adhere to the Dual-Panel contract, sticky-bar header structure, and history back-button requirements?
