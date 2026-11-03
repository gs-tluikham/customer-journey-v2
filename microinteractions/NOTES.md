# Microinteractions lab — change log

This file lists every change made on top of the main prototype for the microinteractions
experiment. **Nothing here touches the main published page** (`/index.html` one level up) —
all of it lives only in `microinteractions/index.html`.

Every new line of code in `index.html` is tagged `MI:` in a comment, and the whole batch of new
CSS/HTML/JS lives in three clearly fenced blocks (search for `MICROINTERACTIONS LAB`):

1. A `<style>` block, inserted right after the existing `.ovsep` rule.
2. A `<script>` block, inserted right after `let LAYOUT='split';`, containing the `MI` config
   object and every new helper function.

(v1 also added a toolbar `<button id="btnGoal">` HUD chip before `#btnCfg`; it was removed
entirely after feedback that it was redundant with the renewal's own flag — see "Renewal-date
gamification" below.)

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
| 2 | Lane transition | `laneTransition` | **v2 (revised after feedback — v1 "still felt like a jump")**: a genuine two-layer cross-fade. A static, click-through clone of the chart's last rendered frame sits on top while the real chart rebuilds underneath (instant, but hidden under the clone), then the clone fades away to reveal the already-finished new chart. New content is on screen the whole time, so there's no blank gap — which is what v1's fade-to-blank-then-fade-in read as a cut. | `miTransitionBuild()`; CSS classes `.mi-lane-snap` / `.mi-lane-snap.mi-fade-out`. Called from the `.tchip` click handler and `layoutSel.onchange` (both previously called `build()` directly). |
| 3 | Page/zoom glide | `pageGlide` | When paging through months or zooming re-buckets the data, markers animate (280ms) to their new bucketed position instead of snapping. The month-span animation itself (`setExtremes`) was already animated before this lab — this adds motion to the *re-bucket* step that follows it. | `refreshLanes(c)` — `animation` arg on the three `series[i].setData(...)` calls, and `c.redraw(MI.pageGlide)`. |
| 4 | "+N" group spring | `groupSpring` | The key-date group popup list springs open (scale .85→1, overshoot easing, 280ms) instead of appearing instantly. | CSS `.mlist.mi-spring` + `@keyframes miSpring`; class added/removed in `showMiniList()` right after `mList.classList.add('show')` (removed on every hide, so it always replays on reopen). |
| 5 | Popover rise | `popoverRise` | The detail popover fades up 6px over 180ms when it opens. | CSS `.pop.mi-rise` + `@keyframes miFadeRise`; class removed-then-re-added (forces a reflow) in `openDetail()` right after `pop.style.display='block'`, so repeated opens always replay it. |
| 5b | Drawer fade | `drawerFade` | The full-screen drawer backdrop (`#edMask`) fades in/out instead of appearing/disappearing instantly. (The drawer panel itself already slid in via `transform:translateX` before this lab — this only adds the backdrop fade.) | CSS `.edmask.mi-fade{opacity:...}` / `.edmask.mi-fade.show{opacity:1}`; the `mi-fade` class is added once at init, right after `const edMask=document.getElementById('edMask')`, so no changes were needed in `openEventDetail()` / `closeEventDetail()` — they already just toggle `.show`. |
| 6 | Key-date flag wave | `flagWave` | A short ripple ring expands and fades (560ms) from a key-date flag when you hover it (grouped flags, which open the mini-list instead, are excluded). **Fixed after feedback**: the ripple was missing `pointer-events:none`, so it sat over the marker and stole the mouseover/mouseout pair from the point underneath as it grew — that's what made the native tooltip "blink." Now set on the ring, same as every other chart overlay (`kdPoles`, `laneLbl`, the TODAY-pulse ring, etc.). | `miFlagWave(point)`; called from `plotOptions.scatter.point.events.mouseOver` when `this.series.name==='Key dates'` and the point isn't a group. |
| 7 | Today-line pulse | `todayPulse` | A soft ring pulses once (900ms) at the TODAY line's top the first time a given chart instance renders — not on every redraw/resize/toggle, only once per `build()`. | Inside the render hook's existing TODAY-line block, guarded by `c.miTodayPulsed` (set `true` the first time it fires) so it can't replay on the same chart instance. |

### Renewal-date gamification (v2 — revised after feedback)

Flag: `renewalGoal`. **v1 put this in a separate toolbar chip (`#btnGoal`); feedback was that
the chip was redundant with the flag already marking the renewal in the key-date lane, and
that it should stay a key date, visually distinct, with more microinteraction on the marker
itself.** The chip (CSS, HTML, JS) has been removed entirely. Everything now lives on the
renewal's own flag, in the key-date lane, where it already was:

- **Larger flag**: still radius 7.5 instead of the ordinary 6 (`markerFor()`), unchanged from v1.
- **Progress ring** (new): a plain SVG ring drawn around the flag — a faint full "track" circle
  plus a solid arc on top that sweeps clockwise from 12 o'clock to **% of the current contract
  term elapsed** (`renewalInfo()`, same calculation as v1's progress bar, just relocated).
  This is the same visual language as an Apple Watch activity ring or a GitHub
  contribution/streak ring — a well-worn "goal progress" pattern. `miArcPath()` builds the arc's
  SVG path.
- **Soft "ping"** (`.mi-goal-halo`, renamed in spirit from v1's flat halo dot): a small circle
  that continuously pulses outward and fades, the same visual grammar as a map's "you are here"
  live-location dot or a notification badge ping — a live/important-location cue sitting
  *behind* the ring, not a progress indicator itself.
- **Twinkling star badge** (new, `.mi-goal-star`): a small gold sparkle (reusing the same
  4-point `SPARK` path already used elsewhere in the UI) positioned at the flag's upper-right,
  with a slow opacity twinkle. Reads the way a starred calendar event or a game quest-marker
  reads at a glance — "this one's the goal."
- **Springier hover pop** (new): the goal flag gets a bigger, bouncier grow on hover
  (`Highcharts.easing.miSpring`, a classic ease-out-elastic) instead of the ordinary
  `hoverSpotlight` grow every other marker gets — wired inside `miSpotlight()` itself (checks
  whether the hovered point is the renewal), not as a separate competing animation.
- **Native tooltip enrichment** (new): hovering the flag now shows "…planned • 129d to go •
  72% of term elapsed" right in the existing tooltip — recovering the chip's informational
  value without a second always-on control.
- **"Jump to this month" in its own popover** (new): clicking the renewal flag opens the normal
  key-date popover, now with a "Jump to this month →" link — recovering the chip's click
  action, contextually, only where the renewal itself is.

**Positioning note**: all three drawn pieces (ping, ring, star) are positioned off the
*actual rendered point* — found by checking each key-date point's bucket contents for the
renewal moment, not by reading a point's own `mi` field (which only names a cluster's
dominant item when the renewal is grouped into a "+N"). A bucket-contents search finds the
right visual marker either way, at its real tier row, even when the lane is crowded enough
to stack key dates onto more than one row.

This is still a first pass for review — ideas for a possible next round, not yet built:
a small celebratory animation once the term is e.g. >90% elapsed, restyling the existing
edge-of-window "Renewal in N mo" pill (`renewPillShown()` / `positionRenewPill()`, left
untouched throughout) to match this visual language, and/or a literal checkered "finish
line" treatment on the flag's pennant itself (closer to a real-world race/finish-line flag)
if the star+ring combination doesn't feel distinct enough in practice.

## Known limitation

`build()` fully destroys and recreates the Highcharts chart on every data/layout/filter
change (this was true before this lab, not something introduced here). That means lane
transitions can't be a true per-marker morph between the old and new layout — `miTransitionBuild()`
disguises the destroy/recreate as a cross-fade rather than actually animating markers from
their old position to their new one. Good enough to remove the "jump", not a real morph.
