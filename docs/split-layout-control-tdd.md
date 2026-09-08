# Split Layout Control — Capability-Gated TDD Addendum

This checklist covers preferred native full-screen split allocation for recurring SpaceDress pair rules.

These slices use the stable `SL-###` prefix. They are capability-gated because macOS does not publish a dedicated API for setting the divider percentage of another app pair's native full-screen Split View. The first slices therefore prove the mechanism before product UI depends on it.

Use the Red → Green → Refactor discipline from `IMPLEMENTATION_PLAN.md`.

A useful prompt wrapper is:

```text
Implement only SpaceDress split-layout slice SL-### using strict TDD.
Read AGENTS.md, docs/user-configuration-model.md, ADR 0005, ADR 0006, and ADR 0007 first.
Start with the failing test/probe, implement only enough to pass it, and do not begin another slice.
```

## A — Capability proof

- [ ] **SL-001 — Probe AX mutability for native split participants.** Red: real-Mac probe creates a native two-app full-screen split and records whether each participant AX window exposes settable position/size attributes, both active and inactive where observable. Green: implement the probe and commit sanitized results with macOS build/application metadata.
- [ ] **SL-002 — Attempt controlled AX split resize.** Red: system test requests a materially different ratio such as 65/35 and fails until the native split actually changes without leaving full-screen Split View or corrupting either participant. Green: implement the smallest supported AX mutation experiment and immediately re-observe geometry.
- [ ] **SL-003 — Characterize native clamping and minimum sizes.** Red: parameterized system test requests several ratios (for example 20/80, 35/65, 50/50, 65/35, 80/20) across controlled app pairs and records achieved geometry. Green: define tolerance/clamping diagnostics rather than retrying indefinitely.
- [ ] **SL-004 — Decide production mutation strategy.** Red: architecture check fails until current evidence selects supported AX, another system-mediated path, or explicitly marks ratio control unavailable. Green: document the selected strategy and capability criteria; private SkyLight mutation requires a separate ADR.

## B — Pair layout model

- [ ] **SL-005 — Add member-relative preferred allocation.** Red: model tests prove a canonical unordered `Chrome+iTerm2` pair can persist `Chrome=0.65, iTerm2=0.35` and still resolve correctly after physical sides swap. Green: add the smallest private user-config model.
- [ ] **SL-006 — Add same-app physical allocation fallback.** Red: tests for `Chrome+Chrome` with no stable instance selectors preserve `leftShare=0.65` without pretending one Chrome window has durable identity. Green: implement physical-side fallback.
- [ ] **SL-007 — Validate allocation values.** Red: tests reject NaN, negative, >1, noncomplementary/ambiguous member allocations, and unusably extreme values before platform clamping. Green: normalize the persisted representation and diagnostics.

## C — Restore behavior

- [ ] **SL-008 — Implement one-shot pair-layout restoration.** Red: integration tests observe a stable matching pair at the wrong ratio, issue exactly one restore attempt, re-observe geometry, and then stop. Green: add capability-gated restore coordinator.
- [ ] **SL-009 — Respect manual divider changes after restoration.** Red: after successful restore, simulate/reobserve a user-changed ratio and assert SpaceDress does not snap it back during the same pair lifetime. Green: track the pair-generation restore state rather than continuously enforcing a target.
- [ ] **SL-010 — Restore again when the pair is recreated.** Red: close/recreate the split pair and verify the preferred allocation is applied to the new pair generation even if native Space IDs and window IDs changed. Green: key behavior to logical pair observation/reconciliation, not native identifiers.
- [ ] **SL-011 — Surface unavailable/clamped/error states.** Red: resolver/UI tests cover unsupported AX mutation, permission denial, timeout, app minimum-width clamp, partial result, and successful exact/tolerant result. Green: expose concise capability/diagnostic state without blocking appearance styling.

## D — UX and acceptance

- [ ] **SL-012 — Add pair layout editor.** Red: UI tests edit a split pair's allocation by app percentage, show 65/35 as complementary values, support reset/restore-now, and fall back to Left/Right terminology for indistinguishable same-app members. Green: integrate into `Customize This Split Pair` without exposing native IDs.
- [ ] **SL-013 — Add direct `Remember Current Split` action.** Red: context-flow test captures the currently observed ratio into the matching pair rule and correctly associates shares with distinct app members regardless of which is left. Green: implement one-action capture from live geometry.
- [ ] **SL-014 — Accept recurring Chrome + iTerm2 split restoration.** Red: real-Mac acceptance creates the configured pair, changes/recreates it multiple times including swapped sides, and expects the intended app-relative allocation within tolerance whenever the capability is declared available. Green: fix only layout-model/restoration defects.

## Release relationship

This feature is **capability-gated**, not a reason to lie about platform support.

- If SL-001 through SL-004 establish a reliable supported production mechanism on a declared macOS configuration, SL-005 through SL-014 become required for claiming split-layout restoration support there.
- If no safe mechanism is available, SpaceDress may ship appearance/instance/pair styling while reporting split-layout restoration unavailable on that platform.
- A future move to private window mutation requires a new explicit ADR rather than silently escalating implementation invasiveness.
