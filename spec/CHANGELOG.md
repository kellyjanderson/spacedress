# Desktop Switcher Appearance Specification changelog

DSAS is versioned independently of SpaceDress.

## 0.1 — Draft

Initial experimental specification:

- neutral **Desktop Switcher Appearance Specification (DSAS)** name;
- **Desktop Switcher Appearance Manifest (DSAM)** document terminology;
- explicit separation from SpaceDress's internal multi-app user manifest;
- renderer-derived application baselines are normal when no DSAM exists;
- macOS app-bundle discovery through `DesktopSwitcherAppearanceManifest` or the conventional `Contents/Resources/DesktopSwitcherAppearance/manifest.json` path;
- ordered declarative layers;
- border and tint layers;
- icon layer using actual bundle icon or local resource;
- app title, document title, and literal text sources;
- local image overlays;
- participant-local multi-app/split semantics;
- local-resource containment;
- extension namespace escape hatch;
- JSON Schema and examples.
