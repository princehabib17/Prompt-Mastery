---
name: emil-kowalski-ui-polish
description: >
  Build and review frontend UI motion and interaction detail with Emil
  Kowalski's taste (creator of sonner, vaul, and the "Animations on the Web"
  course): every animation has a purpose, transform/opacity only, spring-like
  easing, interruptible transitions, a small shared set of duration/easing
  tokens instead of per-component values, and directional motion tied to a
  trigger's origin for toasts/drawers/dialogs/popovers. Use this whenever
  building or reviewing a toast, dialog, drawer, sheet, dropdown, tooltip,
  modal, hover card, page transition, or any CSS/JS animation or transition;
  when the user asks for something to "feel smooth", "feel native", "feel
  premium/polished", or complains an animation "feels janky", "snaps", or
  "looks cheap"; or when they mention Framer Motion, React Spring, CSS
  transitions/keyframes, or `prefers-reduced-motion`.
---

# emil-kowalski-ui-polish

Motion is a communication tool, not a decoration budget. Every animation in
a good interface is answering one of: "where did this come from", "what
just changed", or "what's more important right now". If an animation isn't
answering one of those, cut it.

## Core moves, in priority order

### 1. Motion needs a job

Before adding any animation, name what it's communicating. If the honest
answer is "it looked cool in a video", don't add it.

- A dialog scaling up from 95% while fading in says "this appeared, and it
  came from roughly here."
- A list item sliding out before its neighbors slide up says "this was
  removed, here's what filled the gap."
- A button's press state (`scale: 0.97`) says "your click registered."

If two animations on the same screen are communicating the same thing
twice, keep the more useful one and drop the other.

### 2. CSS first, JS only when you need real physics or interruption logic

CSS transitions and `@keyframes` run off the main thread for
`transform`/`opacity` and are the right default for anything simple:
hovers, fades, position shifts driven by a class toggle. Reach for a JS
animation library (Framer Motion, React Spring, GSAP) only when the
animation needs something CSS genuinely can't do cleanly — physically
accurate spring interruption, gesture-driven dragging (a drawer following
a finger), or animating between two arbitrary layout states (shared-layout
transitions).

```css
/* A hover/press micro-interaction needs nothing more than this */
.button {
  transition: transform 150ms cubic-bezier(0.2, 0, 0, 1);
}
.button:active {
  transform: scale(0.97);
}
```

### 3. Animate `transform` and `opacity` — never layout properties

`width`, `height`, `top`, `left`, `margin`, and similar trigger layout
recalculation on every frame, which is where jank comes from. `transform`
and `opacity` are composited and can hit 60fps+ on cheap hardware.

```css
/* Avoid — triggers layout every frame */
.panel {
  transition: width 200ms, left 200ms;
}

/* Prefer — same visual result, composited */
.panel {
  transition: transform 200ms;
  transform: translateX(var(--offset));
}
```

`clip-path` is the exception worth knowing: it's compositor-friendly and
is the right tool for wipes/reveals that would otherwise need a layout
property.

### 4. Respect `prefers-reduced-motion`

This isn't an afterthought pass at the end — bake it into whatever token
system or animation primitive the project uses, once, so every animation
built on top of it inherits the behavior for free.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

For JS-driven animation (Framer Motion etc.), read the same media query
and either skip the animated variant or collapse duration to near-zero —
don't just disable the feature the animation was communicating.

### 5. Spring-like easing, tuned durations

Linear and the browser default `ease` read as mechanical. Real physical
motion accelerates and decelerates — reach for a spring, or at minimum an
easing curve shaped like one (fast start, gentle settle).

Rough duration guide:

| What's moving | Duration |
|---|---|
| Micro-interaction (button press, checkbox, icon swap) | 100–200ms |
| Small element enter/exit (tooltip, dropdown item) | 150–250ms |
| Medium surface (toast, popover, dropdown menu) | 200–300ms |
| Large surface (dialog, drawer, sheet) | 250–400ms |
| Page-level transition | 300–500ms |

A spring config that reads as "natural" for most UI (Framer Motion):

```ts
transition: { type: "spring", stiffness: 400, damping: 30 }
```

Softer/bouncier for playful UI, stiffer/heavier-damped for a dense
productivity app — but pick one feel and reuse it, don't retune per
component (see token point below).

### 6. Interruptible, not snap-or-restart

A hover-triggered animation that gets interrupted mid-flight by a
hover-out should smoothly reverse from its current position, not jump to
the end state and then animate back, and not ignore the interruption and
finish the original animation first.

CSS transitions get this for free automatically — toggling the class
back mid-transition reverses from the current computed value. This is
one of the strongest reasons to prefer CSS transitions over
`@keyframes`/JS animations for anything that responds to fast, repeated
user input (hover, drag), where `@keyframes` restarts from 0% and a naive
JS animation library can also restart rather than reverse if not
explicitly configured for interruption.

If using a JS library, confirm it interrupts correctly by rapid-toggling
the trigger — Framer Motion and React Spring both handle this correctly
by default when animating from current velocity, but a custom
`setTimeout`-based animation typically does not.

### 7. A shared token set, not per-component values

Every one-off `transition: all 0.3s ease-in-out` scattered through a
codebase is a decision nobody made twice, and it's why a product ends up
feeling inconsistent even when each individual component "works." Define
a handful of durations and one or two easing curves as tokens, and pull
every component's motion from that set.

```css
:root {
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --duration-fast: 150ms;
  --duration-base: 250ms;
  --duration-slow: 400ms;
}
```

Never `transition: all` — it animates properties you didn't intend to
(including ones that will change unexpectedly later) and makes the
actual intent of the transition illegible. Name the specific properties.

### 8. Directional motion tied to origin

Toasts, drawers, dialogs, and popovers should enter and exit along the
axis that connects them to whatever triggered them, not a generic
fade-in-place. A toast stacking from the bottom-right should slide up
from that corner; a drawer opened from the right edge should slide in
from the right; a dropdown anchored below a button should scale from
that button's position (`transform-origin`), not from the panel's own
center.

Exit motion should mirror entrance motion in reverse, not switch to a
plain fade — an element that entered by sliding in from the right should
generally leave by sliding back out to the right, so the motion reads as
one continuous, reversible action rather than two unrelated effects.

### 9. Restraint

Fewer, well-tuned animations beat many mediocre ones. A screen with ten
different easing curves and durations feels busier and less polished than
a screen with two elements animated really well and everything else
appearing instantly. When in doubt about whether an animation earns its
place, cut it and see if anything is actually lost.

### 10. Focus states are part of polish, not a separate a11y pass

A keyboard-focused element needs a visible, animatable focus ring that
follows the same motion language as everything else (a quick
transform/opacity fade-in, not an instant hard outline that appears and
disappears with no transition). Treat `:focus-visible` styling as a
first-class interaction state to design, not a compliance checkbox added
at the end.

## Review checklist

When reviewing UI/animation code rather than writing it, scan for:

- [ ] Any animation whose purpose you can't state in one sentence
- [ ] `transition`/`animation` touching `width`, `height`, `top`, `left`,
      `margin`, or other layout-triggering properties instead of
      `transform`
- [ ] `transition: all` instead of naming specific properties
- [ ] No `prefers-reduced-motion` handling anywhere in the codebase
- [ ] Linear or default `ease` timing on anything meant to feel physical
- [ ] Duration outliers — a micro-interaction taking >300ms, or a dialog
      snapping in under 100ms
- [ ] Hover/press animations that snap or restart instead of reversing
      smoothly when interrupted mid-transition
- [ ] Ad hoc duration/easing values sprinkled per-component instead of
      pulled from a shared token set
- [ ] A toast/drawer/dialog/popover that fades in place instead of moving
      along the axis toward its trigger, or whose exit doesn't mirror its
      entrance
- [ ] Missing or non-animated `:focus-visible` styles
- [ ] More distinct animation "vocabularies" on one screen than a user
      could describe back after looking at it once

Don't add animation to hit this checklist — use it to catch motion that's
either doing the wrong job or is technically sloppy in an animation that
already earns its place.
