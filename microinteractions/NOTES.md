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

## Dropped from the earlier (v3-based) lab

The earlier lab's own renewal chip (lavender pennant pill, goal card, progress copy) is gone;
the final build's renewal chip is used instead. Its flag wiggle, flag ripple, TODAY ring,
group spring and popover rise are gone because the final build already covers each of them.
