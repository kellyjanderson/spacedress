# 0003 — Overlay before injection

**Status:** Accepted  
**Date:** 2026-09-07

## Context

macOS provides no supported API for changing Mission Control's Space-thumbnail chrome. Historically, deeper customization has been possible by injecting into system processes or using mechanisms that require weakened SIP.

Modern open-source utilities also demonstrate a less invasive path: observe Mission Control and place transparent application-owned overlays aligned with its UI.

The desired SpaceDress features—icons, titles, borders, tints, and bitmap overlays—do not intrinsically require owning Apple's renderer.

## Decision

SpaceDress will implement visual decoration as application-owned overlays before considering any form of Dock/Mission Control code injection.

Normal installation and use must not require disabling or weakening SIP.

Private read-only SkyLight/CGS data is allowed behind adapters. Private system-process injection is outside the standard architecture.

## Consequences

- SpaceDress cannot directly change native thumbnail spacing or internal Mission Control layout.
- Geometry tracking becomes an important compatibility problem.
- The project remains substantially safer to install and easier to distribute directly.
- Visual features should be designed so an overlay can express them.

## Alternatives considered

### Inject into the Dock

Provides deeper control but increases fragility, security risk, release burden, and user setup cost. Rejected for the core product.

### Replace Mission Control

Would grant complete layout control but discards the mature native interaction users already know. Rejected.

## Revisit when

Only if Apple introduces a supported extension point or equivalent mechanism that provides native decoration without compromising system integrity.
