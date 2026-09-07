# SpaceDress Roadmap

The roadmap is ordered by **risk retirement**, not by visual excitement. SpaceDress should prove it can identify and decorate full-screen Spaces reliably before building a large preference UI.

## Phase 0 — Measure the closet

Goal: establish repeatable macOS capability probes with SIP enabled.

- [ ] enumerate managed Spaces per display;
- [ ] distinguish regular Desktop, single-app full-screen, and split/tiled full-screen Spaces;
- [ ] resolve full-screen owner PID(s) where possible;
- [ ] map PID → `NSRunningApplication` → bundle URL, bundle identifier, localized name, and icon;
- [ ] verify behavior with multiple copies of an app bundle in different folders;
- [ ] verify multiple running instances of the same bundle;
- [ ] identify Mission Control open/close transitions;
- [ ] determine thumbnail frames across display sizes, scaling modes, and macOS versions;
- [ ] prove a click-through overlay can remain visually aligned during Mission Control animation;
- [ ] record failure modes when private data is unavailable;
- [ ] build a compatibility matrix for current supported macOS releases.

**Exit criterion:** a diagnostic tool can print a trustworthy full-screen Space snapshot and a prototype can place one stable decoration on the correct thumbnail.

## Phase 1 — First fitting

Goal: solve the original single-app full-screen recognition problem.

- [ ] application icon overlay;
- [ ] app title overlay;
- [ ] configurable border;
- [ ] configurable tint;
- [ ] click-through rendering;
- [ ] user enable/disable per app;
- [ ] no persistent dependency on native Space identifiers;
- [ ] graceful fallback to app name/icon when richer identity fails.

**Exit criterion:** a user with many dark-mode full-screen apps can identify the desired Space at a glance.

## Phase 2 — Two-piece suit

Goal: handle split/tiled full-screen as a first-class model.

- [ ] resolve both participants;
- [ ] preserve participant ordering/geometry when available;
- [ ] render both app icons without ambiguity;
- [ ] define layer clipping/composition rules;
- [ ] test same-app split configurations;
- [ ] test participants from duplicate bundle identifiers/locations.

**Exit criterion:** split Spaces are at least as easy to identify as single-app full-screen Spaces.

## Phase 3 — The SpaceDress Style Manifest

Goal: let applications ship preferred styling without depending on SpaceDress internals.

- [ ] finalize manifest discovery inside `.app` bundles;
- [ ] validate manifest schema;
- [ ] bundle-icon source;
- [ ] app-title and document-title sources;
- [ ] border, tint, icon, text, and bitmap image layers;
- [ ] strict local-resource sandboxing;
- [ ] unknown-field/version behavior;
- [ ] user override precedence;
- [ ] manifest diagnostics and validation tool;
- [ ] publish compatibility examples.

**Exit criterion:** an unrelated macOS app can add a declarative resource to its bundle and reliably influence its SpaceDress thumbnail appearance.

## Phase 4 — Wardrobe

Goal: polished end-user configuration.

- [ ] visual style editor;
- [ ] per-app rules;
- [ ] per-document rules where stable identity exists;
- [ ] reusable user style packages;
- [ ] preview without entering Mission Control;
- [ ] export/import styles;
- [ ] accessibility controls for contrast, motion, and transparency;
- [ ] privacy control for document titles.

## Phase 5 — Regular Desktops, where useful

Regular Desktop Spaces may receive:

- [ ] user-defined names;
- [ ] representative app icons;
- [ ] reusable style rules;
- [ ] app-set summaries.

This phase must not force the full-screen identity model into a weaker index-based design.

## Explicitly not on the critical path

- replacing Mission Control;
- moving, creating, deleting, or reordering Spaces;
- changing native thumbnail spacing;
- Dock/WindowServer injection;
- requiring SIP to be disabled;
- cloud accounts or synchronization;
- telemetry as a prerequisite for operation.

These can be revisited only if they become independently compelling and respect the project's security boundaries.
