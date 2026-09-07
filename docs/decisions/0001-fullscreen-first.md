# 0001 — Full-screen first

**Status:** Accepted  
**Date:** 2026-09-07

## Context

The motivating problem is not generic Space management. It is the visual similarity of native full-screen app thumbnails in Mission Control, especially when multiple applications use dark themes.

Regular Desktop Spaces have a different identity problem because they can contain many applications and windows without a single privileged owner.

Trying to solve both equally from the beginning would encourage weak abstractions such as identifying every Space by index or choosing an arbitrary "representative" desktop app.

## Decision

SpaceDress treats single-app full-screen and split/tiled full-screen Spaces as its primary product domain.

Priority:

1. single-app full-screen;
2. split/tiled full-screen;
3. regular Desktop Spaces where a sound model exists.

Architecture, identity, performance, and UI decisions are evaluated against full-screen behavior first.

## Consequences

- Owner application identity is a foundational concept.
- Split full-screen participants must be preserved rather than flattened.
- Regular desktops may initially receive limited or no decoration.
- A design that improves regular desktops but weakens full-screen identity can be rejected.

## Alternatives considered

### General Space renamer/manager first

Rejected because it solves a broader but different problem and encourages persistence by desktop order/ID.

### Full replacement switcher

Rejected because the native Mission Control thumbnail already provides valuable live context; SpaceDress only needs to make it distinguishable.

## Revisit when

Revisit only after full-screen behavior is robust and regular Desktop users present a concrete identity model that does not compromise it.
