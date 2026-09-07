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
- Style manifests are declarative. Never add scripts, commands, dynamic libraries, remote JavaScript, or equivalent executable hooks to the manifest format.
- No telemetry or network dependency without an explicit product decision.

## macOS claims

Private macOS behavior changes. When working on platform integration:

- state the exact macOS version/build tested;
- distinguish public API, private SPI, preference-file observation, and heuristic inference;
- capture raw observations in a reproducible probe or research note;
- do not infer persistence from seeing the same identifier twice;
- do not infer ownership from thumbnail order alone;
- test single full-screen and split/tiled full-screen separately;
- test multi-display behavior with both "Displays have separate Spaces" states when relevant.

## Implementation shape

Domain code should consume stable project-owned models such as `SpaceSnapshot`, `SpaceParticipant`, `ApplicationIdentity`, and `ResolvedStyle` rather than dictionaries returned by SkyLight or Accessibility.

Platform code should translate unstable system data into those models and attach provenance/capability information when ambiguity matters.

A missing private symbol should produce a capability downgrade, not a crash.

## Swift style

Follow `docs/code-style.md`. Prefer Apple frameworks and the standard library. Keep UI actor isolation explicit. Avoid global mutable state. Make cancellation and lifecycle ownership visible.

## Privacy

Document titles and paths may be sensitive. Do not put them in routine logs, crash metadata, fixtures, screenshots, or bug-report bundles without deliberate redaction.

## Changes to the public manifest

Read `spec/README.md` and update:

- the prose specification;
- JSON Schema;
- at least one example;
- compatibility notes;
- an ADR if semantics or security boundaries materially change.

## Documentation tone

Use the wardrobe/dress metaphor lightly for navigation and memorable names. Technical statements should stay literal and precise.

A good rule: headings may wear a jacket; invariants should not.
