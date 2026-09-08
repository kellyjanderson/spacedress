# Pair and Instance Styling — Required TDD Addendum

This checklist adds the app-pair and per-instance styling requirements discovered after the main `IMPLEMENTATION_PLAN.md` was numbered.

These slices are **required before the first production release**. They use the stable `SI-###` prefix so the existing `SD-###` identifiers do not have to be renumbered.

Use the same strict Red → Green → Refactor discipline defined in `IMPLEMENTATION_PLAN.md`.

A useful prompt wrapper is:

```text
Implement only SpaceDress pair/instance styling slice SI-### using strict TDD.
Read AGENTS.md, docs/user-configuration-model.md, docs/space-model.md, and ADR 0006 first.
Start with the failing test/probe, implement only enough to pass it, refactor with tests green,
and do not start another SI or SD slice.
```

## A — Selector model

- [ ] **SI-001 — Make bundle ID the default application customization key.** Red: selector tests show two observations of the same normal bundle match one app rule while a different bundle ID does not. Green: implement the minimal bundle-ID selector semantics without requiring path/signing fields.
- [ ] **SI-002 — Preserve optional exact-copy refinement.** Red: tests show two app copies sharing a bundle ID match the generic rule but can be separated by an optional canonical bundle-path selector. Green: implement progressive specificity.
- [ ] **SI-003 — Implement canonical unordered pair selectors.** Red: tests prove `A+B == B+A`, `A+A` is valid, and pair identity does not change when physical sides swap. Green: canonicalize the pair as an unordered multiset of application selectors.
- [ ] **SI-004 — Implement pair-rule matching specificity.** Red: tests cover generic bundle-only pairs, exact-copy pair members, same-app pairs, and nonmatches. Green: choose the most specific matching pair deterministically.

## B — Runtime instance model

- [ ] **SI-005 — Define current participant-instance identity.** Red: tests prove two windows from the same bundle/process remain distinct runtime instances by participant/window observation. Green: add the project-owned runtime key without persistence semantics.
- [ ] **SI-006 — Prove PID alone cannot collapse window instances.** Red: fixture with one PID and three window IDs must produce three independently styleable instances. Green: remove any process-level instance assumption.
- [ ] **SI-007 — Add durable document/instance selector boundary.** Red: tests accept a stable document URL/identifier and reject plain volatile window title as durable identity. Green: add an explicit durable-selector type.
- [ ] **SI-008 — Add current-instance binding.** Red: tests assign a style to one runtime participant and verify siblings from the same app remain unchanged. Green: implement a non-durable current-instance rule/lease.

## C — Reusable instance styles and automatic assignment

- [ ] **SI-009 — Add per-app instance style sets.** Red: tests define several named styles under one app selector and validate references/duplicates. Green: add the private user-config model.
- [ ] **SI-010 — Implement manual instance style assignment.** Red: tests choose one style for one current participant and verify it overrides app/pair defaults only for that instance. Green: implement the assignment.
- [ ] **SI-011 — Implement sequential style leasing.** Red: five concurrent same-app participant fixtures receive style slots 1–5 in first-observed lease order, independent of Mission Control ordering. Green: implement the allocator.
- [ ] **SI-012 — Preserve leases across snapshot churn.** Red: reorder Spaces, reshuffle provider arrays, and temporarily miss/reobserve windows without changing surviving instance styles. Green: connect leases to participant reconciliation rather than list position.
- [ ] **SI-013 — Release and reuse a style slot when an instance really closes.** Red: close one participant, pass the configured reconciliation grace period, create a new participant, and expect the released slot to become available. Green: implement lease lifecycle.
- [ ] **SI-014 — Define style-pool overflow behavior.** Red: create more concurrent instances than configured styles and require every thumbnail to remain distinguishable according to the documented fallback. Green: implement ordinal badge or deterministic secondary variation rather than silent duplicate appearance.
- [ ] **SI-015 — Separate runtime lease cache from durable user manifest.** Red: persistence tests fail if PID/window ID leaks into durable configuration while a session-cache round trip may retain live assignments. Green: enforce separate stores/types.
- [ ] **SI-016 — Reconstruct live leases after SpaceDress restart.** Red: fixture/system test restarts SpaceDress while app windows remain live and expects confident current-window mappings to recover without promoting them to durable identity. Green: restore from runtime observations/cache where safe.

## D — Precedence and split-pair composition

- [ ] **SI-017 — Implement user-style specificity precedence.** Red: tests prove `baseline → DSAM → app rule → pair-member rule → explicit instance/document/style-slot rule → required safety policy`. Green: implement deterministic merge precedence.
- [ ] **SI-018 — Add pair-level container appearance.** Red: split render tests apply an outer pair border/title without erasing independently resolved participant styles. Green: add container-level composition separate from participant appearance.
- [ ] **SI-019 — Add pair-member contextual overrides.** Red: tests let Chrome receive one treatment when paired with VS Code while retaining an explicit instance style as the more specific choice. Green: implement contextual member overrides.
- [ ] **SI-020 — Support same-app pair member differentiation by geometry.** Red: `Chrome+Chrome` split tests apply current left/right contextual presentation only after geometry is known, without making left/right part of persistent pair identity. Green: implement geometry-stage role application.

## E — Direct Mission Control customization

- [ ] **SI-021 — Probe passive right-click observation in Mission Control.** Red: real-Mac system probe records `rightMouseDown` screen coordinates while SpaceDress visual overlays remain mouse-transparent and verifies whether Mission Control performs a conflicting native action. Green: document the viable interaction path or required fallback.
- [ ] **SI-022 — Hit-test observed context clicks to Space/participant geometry.** Red: coordinate tests cover single fullscreen, split 50/50, split 70/30, display origins/scales, gaps, and outside clicks. Green: map a passive click to the correct observed participant without native Space ID persistence.
- [ ] **SI-023 — Present instance context UI without replacing Mission Control interaction.** Red: integration/UI test opens a SpaceDress context surface for a recognized participant while ordinary left-click selection remains native. Green: implement the least invasive menu/popover mechanism supported by SI-021 evidence.
- [ ] **SI-024 — Add `Customize This Instance` flow.** Red: UI-to-resolver test chooses a style and immediately updates only the selected participant. Green: wire direct instance editing.
- [ ] **SI-025 — Add quick `Apply Instance Style` submenu.** Red: UI test applies one of the app's reusable style slots with one context action and verifies persistence scope is clearly reported. Green: implement quick selection.
- [ ] **SI-026 — Add `Customize App Defaults` context flow.** Red: selecting it from an instance edits the bundle-ID app rule and affects sibling instances lacking more-specific overrides. Green: route to the existing app editor.
- [ ] **SI-027 — Add `Customize This Split Pair` flow.** Red: split UI test opens a pair-specific editor keyed by the canonical pair selector and verifies the rule continues matching after sides swap. Green: implement pair editing.
- [ ] **SI-028 — Offer durable `Remember for this document` only when justified.** Red: UI tests show the action when a stable document selector exists and hide/disable it for title-only/current-window cases. Green: wire durable instance binding without overstating persistence.

## F — Settings and acceptance

- [ ] **SI-029 — Add per-app instance-style-set editor.** Red: view-model/UI tests create, name, reorder, edit, duplicate, and delete style slots while preserving active assignments where possible. Green: implement settings UI.
- [ ] **SI-030 — Add instance auto-assignment policy control.** Red: tests switch manual/sequential policy and verify allocator behavior plus persisted user configuration. Green: add the control.
- [ ] **SI-031 — Add live instance assignment inspector.** Red: diagnostics/settings fixture shows current participants, assigned style slots, binding scope, and confidence without exposing sensitive document data by default. Green: implement a troubleshooting view.
- [ ] **SI-032 — Accept five visually identical same-app fullscreen windows.** Red: system/fixture acceptance requires five windows sharing bundle/process identity to receive five unambiguous visual treatments and retain them through Mission Control reorder. Green: fix instance allocation/reconciliation defects only.
- [ ] **SI-033 — Accept three same-app document windows with mixed persistence.** Red: test one stable document binding, one manually styled current window, and one auto-assigned slot simultaneously; restart/reorder behavior must match each scope's documented guarantees. Green: fix precedence/persistence defects.
- [ ] **SI-034 — Accept recurring split-pair styling.** Red: create A+B, swap sides, close/recreate the split, and verify the canonical pair rule remains applicable while participant geometry is recomputed live. Green: fix pair identity/composition defects.

## Completion gate

The first production release is not complete until:

- all required `SD-###` production slices in `IMPLEMENTATION_PLAN.md` are complete;
- **SI-001 through SI-034 are complete**;
- the five-instance, mixed-document, and recurring-pair acceptance scenarios above pass on every supported macOS release where the required underlying capability is declared available.
