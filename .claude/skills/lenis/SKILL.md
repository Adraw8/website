---
name: lenis
description: Lenis smooth-scroll library (darkroom.engineering) — setup, options, scrollTo, anchors, nested scroll, reduced motion, GSAP ScrollTrigger sync, React/Vue adapters, and lenis/snap. Use when the user wants smooth scrolling, inertia/momentum scroll, buttery scroll, scroll syncing with GSAP or WebGL, smooth anchor links, or mentions Lenis, ReactLenis, VueLenis, or useLenis.
---

# Lenis — smooth scroll

Lenis wraps the browser's **native** scroll (so `position: sticky`, anchors and accessibility keep working) and smooths wheel/touch input. Built from the official docs at github.com/darkroomengineering/lenis (MIT). Full docs are in `references/`:

- `references/core.md` — every option, property, method and event (read for anything not covered below)
- `references/react.md` — `<ReactLenis>`, `useLenis`, Framer Motion sync
- `references/vue.md` — `<VueLenis>`, plugin registration, `useLenis`
- `references/snap.md` — `lenis/snap` (Lenis does not support CSS scroll-snap)
- `references/lenis.css` — the recommended stylesheet

## Install

```bash
npm i lenis
```
```js
import Lenis from 'lenis'
import 'lenis/dist/lenis.css'
```

No build step (static HTML sites) — pin an exact version:

```html
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.27/dist/lenis.css">
<script src="https://unpkg.com/lenis@1.3.27/dist/lenis.min.js"></script>
<script>
  const lenis = new Lenis({ autoRaf: true, anchors: true, allowNestedScroll: true, autoToggle: true, stopInertiaOnNavigate: true })
</script>
```

## Setup rules

1. **Always include the recommended CSS** — most "it doesn't work" bugs come from missing it.
2. **Drive the loop exactly once:** either `autoRaf: true`, or call `lenis.raf(timeInMs)` every frame yourself. Never both.
3. **With GSAP ScrollTrigger**, let GSAP's ticker drive Lenis:

```js
const lenis = new Lenis()            // no autoRaf
lenis.on('scroll', ScrollTrigger.update)
gsap.ticker.add((time) => lenis.raf(time * 1000))  // GSAP time is seconds; Lenis wants ms
gsap.ticker.lagSmoothing(0)
```

4. **Anchor links** are blocked while smoothing unless `anchors: true` (or `anchors: { offset, onComplete }`).
5. **Nested scrollers** (modals, dropdowns, code blocks): `allowNestedScroll: true` is simplest; for performance prefer `data-lenis-prevent` on the element (or `-wheel`, `-touch`, `-vertical`, `-horizontal` variants) or the `prevent: (node) => ...` option.
6. **Modals / scroll lock:** `lenis.stop()` / `lenis.start()`, or `autoToggle: true` (follows the wrapper's `overflow`; needs the CSS).
7. **Reduced motion** is honored by default (`respectReducedMotion: true`) — leave it on. Check `lenis.prefersReducedMotion` to tone down your own animations too.
8. **Clean up** with `lenis.destroy()` on unmount / page teardown (SPA routes).

## Common options

| Option | Default | Notes |
|---|---|---|
| `lerp` | `0.1` | 0–1 smoothing intensity; overrides `duration`/`easing` |
| `duration` / `easing` | `1.2` / expo-out | Used only if `lerp` is not set |
| `orientation` | `vertical` | `horizontal` for side-scrolling; pair with `gestureOrientation` |
| `smoothWheel` | `true` | |
| `syncTouch` | `false` | Smooth touch too; can be unstable on iOS < 16 |
| `wheelMultiplier` / `touchMultiplier` | `1` | Scroll speed |
| `wrapper` / `content` | `window` / `<html>` | For a custom scroll container |
| `infinite` | `false` | Needs `syncTouch: true` on touch devices |

## Useful API

```js
lenis.scrollTo('#contact', { offset: -80, duration: 1.5 })  // number | selector | element | 'top' | 'bottom'
lenis.scrollTo(0, { immediate: true })
lenis.on('scroll', ({ scroll, velocity, direction, progress }) => {})
lenis.stop(); lenis.start(); lenis.resize(); lenis.destroy()
```

## React / Vue

```jsx
import { ReactLenis, useLenis } from 'lenis/react'
<ReactLenis root options={{ lerp: 0.1 }} />
const lenis = useLenis(({ scroll }) => {})
```
For GSAP in React, pass `options={{ autoRaf: false }}` plus a `ref`, and in an effect add `(t) => ref.current?.lenis?.raf(t * 1000)` to `gsap.ticker` (remove on cleanup). Vue: `<VueLenis root />` + `useLenis` from `lenis/vue`. See references.

## Limitations

- No CSS scroll-snap — use `lenis/snap` (`new Snap(lenis, { type: 'mandatory' })`, `snap.addElements(...)`).
- Capped at 60fps in Safari (30fps in low-power mode).
- Smoothing stops over iframes (they don't forward wheel events).

## Troubleshooting

Latest version? CSS included? Loop running once (`autoRaf` or manual `raf`)? Page scrollable with Lenis removed? ScrollTrigger wired as above?
