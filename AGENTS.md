# Instructions for coding and research agents

This repository is deliberately specification-heavy before implementation. Read the project decisions before producing code.

## Source-of-truth order

When documents disagree, prefer:

1. accepted ADRs in `docs/decisions/`;
2. versioned public specification under `spec/`;
3. architecture and model documents under `docs/`;
4. current implementation behavior;
5. comments and issue discussion.

Do not silently resolve a real contradiction. Surface it and update the appropriate source of truth.

## Non-negotiable project invariants

- Full-screen Spaces are the primary use case.
- Split/tiled full-screen is a first-class case, not an edge case.
- Native Space IDs, UUIDs, indices, and positions are **ephemeral observations** unless a future ADR explicitly proves and narrows a stronger guarantee.
- Do not use native Space identifiers as persistent user identity.
- Do not require users to disable or weaken SIP.
- Do not inject code into the Dock or WindowServer in the normal architecture.
- Keep private SkyLight/CGS/SLS calls behind a platform adapter.
- Prefer read-only private interfaces; private mutation requires an explicit ADR.
- Split participant placement comes from window geometry. Never treat owner/PID/window array order as left/right.
- SpaceDress must derive a useful appearance when an app supplies no integration metadata.
- The public standard is the **Desktop Switcher Appearance Specification (DSAS)**; its document is a **Desktop Switcher Appearance Manifest (DSAM)**.
- DSAS is application-facing and must not absorb SpaceDress branding, targeting rules, or the internal multi-app user manifest.
- DSAM data is declarative. Never add scripts, commands, dynamic libraries, remote JavaScript, or equivalent executable hooks.
- No telemetry or network dependency without an explicit product decision.

## macOS claims

Private macOS behavior changes. When working on platform integration:

- state the exact macOS version/build tested;
- distinguish public API, private SPI, preference-file observation, and heuristic inference;
- capture raw observations in a reproducible probe or research note;
- do not infer persistence from seeing the same identifier twice;
- do not infer ownership or geometry from thumbnail/array order alone;
- record participant window IDs and bounds when testing split/tiled full-screen;
- test unequal split ratios and swapped left/right participants;
- test same-app split/tiled configurations;
- test single full-screen and split/tiled full-screen separately;
- test multi-display behavior with both "Displays have separate Spaces" states when relevant.

## Implementation shape

Domain code should consume stable project-owned models such as `SpaceSnapshot`, `SpaceParticipant`, `ApplicationIdentity`, `ParticipantRegion`, and `ResolvedStyle` rather than dictionaries returned by SkyLight or Accessibility.

Platform code should translate unstable system data into those models and attach provenance/capability information when ambiguity matters.

A missing private symbol should produce a capability downgrade, not a crash.

## Appearance-source separation

Keep these concepts distinct:

```text
running app metadata ──> derived baseline ──> optional DSAM ──> application appearance
                                                                  │
SpaceDress internal multi-app user manifest ──────────────────────┘
                                                                  │
                                                                  v
                                                        resolved appearance
```

The internal SpaceDress user manifest may evolve independently. Do not publish it as DSAS merely because it uses similar visual primitives.

## Swift style

Follow `docs/code-style.md`. Prefer Apple frameworks and the standard library. Keep UI actor isolation explicit. Avoid global mutable state. Make cancellation and lifecycle ownership visible.

## Privacy

Document titles and paths may be sensitive. Do not put them in routine logs, crash metadata, fixtures, screenshots, or bug-report bundles without deliberate redaction.

## Changes to DSAS

Read `spec/README.md` and update:

- the prose specification;
- JSON Schema;
- at least one example;
- compatibility notes;
- the DSAS changelog;
- an ADR if semantics or security boundaries materially change.

Do not change SpaceDress's user configuration format as part of DSAS unless an ADR first establishes a real interoperability need.

## Documentation tone

Use the wardrobe/dress metaphor lightly for navigation and memorable names. Technical statements should stay literal and precise.

A good rule: headings may wear a jacket; invariants should not.
