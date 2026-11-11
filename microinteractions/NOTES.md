# Microinteractions lab — notes

`microinteractions/index.html` is the **final** build (`Customer Journey — final.html`) with a
small set of microinteractions added on top. The main prototype at the repo root is untouched.

## What the final build already does (kept as is)

These were in the final file, so the lab does not add its own version:

- **Customer** picker in the rig bar. *Joint (Staircase + CS)* shows Staircase events and uses
  `ROWS_JOINT` rows; *CS only* drops lifecycle and manual events, hides the Events toggle, and
  uses `ROWS_CS` rows. Height is fixed per customer type, never per account.
- **Renewal chip** on the key-date strip: flag, "Renewal", "in N mo", and an arrow. It drops
  to the compact "in N mo →" form, or lifts above the strip, when a key-date flag would be
  under it (`chipMode`, `positionRenewPill`). On hover the pennant waves and the arrow nudges.
- **Earlier-milestones chip**: a stack of three diamonds that fans out on hover.
- **Key-date flags** wave on their pole when hovered (`.kd-flag`, `cjWave`).
- **TODAY** dot pulses (`today-pulse`).
- **Popovers and lists** fade and rise in (`cjIn`).
- **Health bars**: hovering one fades the other weeks (`focusBar`).

## What the lab adds

All tagged `MI:` and gated by the `MI` flags near the top of the script
(`MI_ENABLED` switches all of them off). Styles are in the "MICROINTERACTIONS LAB" block.

- **hoverSpotlight** — a hovered marker grows and the rest of its lane fades. Key-date flags
  only take the fade (the final build switches their swell off). `miSpotlight` /
  `miClearSpotlight`, wired in `plotOptions.scatter.point.events`.
- **laneTransition** — showing or hiding a type, changing Customer, or changing Scenario
  cross-fades the old frame over the rebuilt chart instead of jumping. `miTransitionBuild`.
- **pageGlide** — paging re-buckets markers and they glide to their new place.
  `refreshLanes`.
- **drawerFade** — the event drawer's backdrop fades in with the slide. `.edmask.mi-fade`.

Fixes carried over (not gated, because they are bugs rather than effects):

- **Health readout clears the key-date card.** It moves above, below, or beside the flag's
  tooltip instead of covering it. `placeHealthTip`.
- **Health hover area matches the bar.** The invisible target is the drawn bar plus 2px, not
  the whole strip plus 4px into the plot, and the readout closes when the pointer moves to a
  marker or anywhere else. A redraw drops the bar without a mouseout, so a document-level
  move check closes it.
- **Earlier-milestones chip is 28px**, matching the paging buttons.

## Expanded view: rows match the data

Fixes to `fitFullscreenLanes` and `chooseEventRows` (these are bugs in final, not effects, so
they are not behind an `MI` flag):

- **No empty rows at the top.** The pitch was stretched to fill the height from the row count
  at the finest grouping. At the wider pitch fewer rows fit, so the grouping coarsened, needed
  fewer rows, and the spare rows sat empty above the last label (burst18 and "Busy events, one
  milestone" used 11 of 16, about 200px blank). The pitch is now the largest one at which the
  lane still holds every row the data asked for, found by measuring `rowCount` rather than by
  the estimate, which ignores the chevron gutter.
- **Finer grouping where there is room.** Because the lane keeps the rows the fit asked for,
  the ladder stays on the finer window: the default account goes from 21 to 22 markers in view
  and busy ones show more individual events instead of +N counts.
- **Collapse returns to the compact height.** The expanded pitch was still set when `build()`
  chose the compact row count, so collapsing left the chart 414px tall instead of 306px.
  `chooseEventRows` now clears it first.

Still true by design: a sparse account is held by `FS_PITCH_MAX` (140px), so a lane with only
a few events leaves its top empty rather than spreading rows further apart. The chevron
gutter in the lane's middle still adds one extra gap in the row ladder.

## Dropped from the earlier (v3-based) lab

The earlier lab's own renewal chip (lavender pennant pill, goal card, progress copy) is gone;
the final build's renewal chip is used instead. Its flag wiggle, flag ripple, TODAY ring,
group spring and popover rise are gone because the final build already covers each of them.
