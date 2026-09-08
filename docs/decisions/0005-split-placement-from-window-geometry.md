# 0005 — Split Placement Comes From Window Geometry

**Status:** Accepted  
**Date:** 2026-09-07

## Context

A tiled/full-screen Space can contain two application participants. Private macOS interfaces may expose owner PIDs or window IDs, but returned array order is not a documented statement that one participant is left and the other right.

SpaceDress needs more than an unordered pair. Correct decoration depends on knowing which participant occupies which part of the thumbnail, including unequal split ratios.

macOS window information can expose screen-space window bounds. Those bounds are a direct geometric observation and can be associated with the participant window and application.

## Decision

SpaceDress treats **participant geometry as the source of truth for split placement**.

For each participant, retain the best available:

```text
windowID
screenBounds
normalizedRegion
geometryConfidence
```

Physical side is derived from geometry as a convenience, never used as the primary representation.

For a conventional two-window horizontal split:

```text
lower midX  → left
higher midX → right
```

If a future arrangement is vertical, equivalent `midY` geometry can derive top/bottom. More complex layouts should remain rectangles/regions rather than being forced into side labels.

Normalize participant bounds relative to the tiled content frame (normally the union of participant bounds). When a Mission Control thumbnail frame is known, transform the normalized regions into thumbnail coordinates.

This preserves native proportions such as 70/30 rather than assuming equal halves.

The following are explicitly **not** placement evidence:

- owner-array order;
- PID order;
- window enumeration order;
- Mission Control index;
- native Space identifier ordering.

When geometry cannot be obtained or corroborated, placement is unknown and the renderer falls back to a neutral multi-participant treatment.

## Consequences

- Left/right should be determinable without a dedicated "left participant" API.
- Unequal split ratios become first-class information.
- The domain model remains ready for top/bottom or more complex arrangements.
- Mission Control composition can clip each participant's decoration to the correct thumbnail region.
- Phase 0 must empirically verify window bounds for native tiled/full-screen Spaces on each supported macOS release.

## Alternatives considered

### Trust owner-array ordering

Rejected because the order is undocumented and may reflect internal bookkeeping rather than geometry.

### Assume the first participant is left and divide the thumbnail 50/50

Rejected because it fails immediately for swapped participants and unequal split ratios.

### Detect left/right from application identity or launch order

Rejected because neither has a geometric relationship to where the user placed the window.

## Revisit when

macOS provides a supported API that directly exposes participant regions with stronger guarantees than window-bound correlation.
