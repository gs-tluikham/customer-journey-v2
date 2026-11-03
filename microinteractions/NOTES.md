# Microinteractions lab — change log

This file lists every change made on top of the main prototype for the microinteractions
experiment. **Nothing here touches the main published page** (`/index.html` one level up) —
all of it lives only in `microinteractions/index.html`.

Every new line of code in `index.html` is tagged `MI:` in a comment, and the whole batch of new
CSS/HTML/JS lives in three clearly fenced blocks (search for `MICROINTERACTIONS LAB`):

1. A `<style>` block, inserted right after the existing `.ovsep` rule.
2. A toolbar `<button id="btnGoal">` (the renewal HUD chip), inserted right before `#btnCfg`.
3. A `<script>` block, inserted right after `let LAYOUT='split';`, containing the `MI` config
   object and every new helper function.

## How to turn effects off

Every effect is gated by a flag in the `MI` object near the top of the script:

```js
const MI_ENABLED = true;   // master switch — set to false to disable every effect at once
const MI = {
  hoverSpotlight: MI_ENABLED && true,
  laneTransition: MI_ENABLED && true,
  pageGlide:      MI_ENABLED && true,
  groupSpring:    MI_ENABLED && true,
  popoverRise:    MI_ENABLED && true,
  drawerFade:     MI_ENABLED && true,
  flagWave:       MI_ENABLED && true,
  todayPulse:     MI_ENABLED && true,
  renewalGoal:    MI_ENABLED && true
};
```

Flip any single flag to `false` to turn that one effect off, or set `MI_ENABLED = false` to
turn all nine off at once, with no other edit needed. All of the gating `if(MI.xxx)` checks
already exist at every call site — the flags are the only thing that needs to change.

## How to remove an effect for good

Search the file for its flag name inside a `MI:` comment (e.g. search `MI: hoverSpotlight`) —
every block tagged with that name is part of that effect. Delete those blocks, plus the
matching rule(s) in the "MICROINTERACTIONS LAB" CSS block if it has one. The table below lists
exactly where each effect's code lives.

## The 7 requested microinteractions + the renewal-goal treatment

| # | Effect | Flag | What it does | Where it's wired |
|---|---|---|---|---|
| 1 | Hover spotlight | `hoverSpotlight` | Hovered marker grows (+2.5px radius); every other marker (and its label) in the **same lane** fades to 30% opacity. Both animate back on mouse-out. | `miSpotlight()` / `miClearSpotlight()` functions; called from `plotOptions.scatter.point.events.mouseOver` / `mouseOut`. |
| 2 | Lane transition | `laneTransition` | Toggling a lane chip or switching Layout no longer jumps straight to the rebuilt chart — `#chart` fades/slides out (130ms), the chart rebuilds while invisible, then fades/slides back in. | `miTransitionBuild()` wraps `build()`; CSS classes `.mi-lane-out` / `.mi-lane-in` on `#chart`. Called from the `.tchip` click handler and `layoutSel.onchange` (both previously called `build()` directly). |
| 3 | Page/zoom glide | `pageGlide` | When paging through months or zooming re-buckets the data, markers animate (280ms) to their new bucketed position instead of snapping. The month-span animation itself (`setExtremes`) was already animated before this lab — this adds motion to the *re-bucket* step that follows it. | `refreshLanes(c)` — `animation` arg on the three `series[i].setData(...)` calls, and `c.redraw(MI.pageGlide)`. |
| 4 | "+N" group spring | `groupSpring` | The key-date group popup list springs open (scale .85→1, overshoot easing, 280ms) instead of appearing instantly. | CSS `.mlist.mi-spring` + `@keyframes miSpring`; class added/removed in `showMiniList()` right after `mList.classList.add('show')` (removed on every hide, so it always replays on reopen). |
| 5 | Popover rise | `popoverRise` | The detail popover fades up 6px over 180ms when it opens. | CSS `.pop.mi-rise` + `@keyframes miFadeRise`; class removed-then-re-added (forces a reflow) in `openDetail()` right after `pop.style.display='block'`, so repeated opens always replay it. |
| 5b | Drawer fade | `drawerFade` | The full-screen drawer backdrop (`#edMask`) fades in/out instead of appearing/disappearing instantly. (The drawer panel itself already slid in via `transform:translateX` before this lab — this only adds the backdrop fade.) | CSS `.edmask.mi-fade{opacity:...}` / `.edmask.mi-fade.show{opacity:1}`; the `mi-fade` class is added once at init, right after `const edMask=document.getElementById('edMask')`, so no changes were needed in `openEventDetail()` / `closeEventDetail()` — they already just toggle `.show`. |
| 6 | Key-date flag wave | `flagWave` | A short ripple ring expands and fades (560ms) from a key-date flag when you hover it (grouped flags, which open the mini-list instead, are excluded). | `miFlagWave(point)`; called from `plotOptions.scatter.point.events.mouseOver` when `this.series.name==='Key dates'` and the point isn't a group. |
| 7 | Today-line pulse | `todayPulse` | A soft ring pulses once (900ms) at the TODAY line's top the first time a given chart instance renders — not on every redraw/resize/toggle, only once per `build()`. | Inside the render hook's existing TODAY-line block, guarded by `c.miTodayPulsed` (set `true` the first time it fires) so it can't replay on the same chart instance. |

### Renewal-date gamification (first pass — not the full idea, see below)

Flag: `renewalGoal`. Treats the renewal as the journey's one goal, with three pieces:

- **HUD chip** (`#btnGoal`, top toolbar, left of the gear icon): a flag icon, "Renewal in
  Nd" (or "Renewal Nd ago" once past), and a small progress bar. The bar reads **% of the
  current contract term elapsed** — computed in `renewalInfo()` from the latest "Contract"
  milestone before the renewal to the renewal date itself (falls back to a flat 365-day
  window if no prior Contract milestone exists, e.g. a brand-new account). Clicking the chip
  jumps/zooms the chart to the renewal's month (same mechanism as clicking a month-axis
  label or a key date).
- **Larger flag**: the renewal's key-date marker renders at radius 7.5 instead of the
  ordinary 6, in `markerFor()`.
- **Pulsing halo**: a soft circle behind the renewal flag that continuously pulses via CSS
  (`.mi-goal-halo` / `@keyframes miGoalPulse`), redrawn every render at the flag's current
  pixel position (same destroy/recreate-per-render pattern as the lane-name labels).

**This is explicitly a first pass for review, not a finished gamification system** — per the
request to "let's see what you can do there first." It intentionally does NOT touch the
existing edge-of-window "Renewal in N mo" pill (`renewPillShown()` / `positionRenewPill()`):
that stays exactly as it was, and the new HUD chip is additive. Ideas for a possible next
round, not yet built: milestones-completed-toward-renewal as a checklist/progress state
(not just elapsed time), a small celebratory animation on the renewal flag once the term
is e.g. >90% elapsed, restyling the existing edge pill to match the chip's visual language,
and/or collapsing the two into one control instead of having both on screen.

## Known limitation

`build()` fully destroys and recreates the Highcharts chart on every data/layout/filter
change (this was true before this lab, not something introduced here). That means lane
transitions can't be a true per-marker morph between the old and new layout — `miTransitionBuild()`
disguises the destroy/recreate as a cross-fade rather than actually animating markers from
their old position to their new one. Good enough to remove the "jump", not a real morph.
