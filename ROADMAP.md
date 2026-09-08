# SpaceDress Roadmap

The roadmap is ordered by **risk retirement**, not by visual excitement. SpaceDress should prove it can identify and decorate full-screen Spaces reliably before building a large preference UI.

For prompt-by-prompt execution, use [`IMPLEMENTATION_PLAN.md`](IMPLEMENTATION_PLAN.md). The roadmap defines product/risk phases; the implementation plan breaks those phases into tracked TDD slices that are intended to fit one coding-agent turn each.

Pair and per-instance styling discovered after the main slice IDs were numbered is tracked in the required pre-v1 [`docs/instance-pair-styling-tdd.md`](docs/instance-pair-styling-tdd.md) addendum rather than renumbering the existing execution plan.

## Phase 0 — Measure the closet

Goal: establish repeatable macOS capability probes with SIP enabled.

- [ ] enumerate managed Spaces per display;
- [ ] distinguish regular Desktop, single-app full-screen, and split/tiled full-screen Spaces;
- [ ] resolve full-screen owner PID(s) where possible;
- [ ] enumerate the actual participant window IDs for each full-screen/tiled Space;
- [ ] obtain each participant window's screen-space bounds;
- [ ] verify split participant geometry can reliably identify physical left/right placement;
- [ ] verify unequal split ratios can be preserved as normalized participant regions;
- [ ] verify geometry with same-app split configurations;
- [ ] map PID → `NSRunningApplication` → bundle URL, bundle identifier, localized name, and icon;
- [ ] verify behavior with multiple copies of an app bundle in different folders;
- [ ] verify multiple running instances of the same bundle;
- [ ] identify Mission Control open/close transitions;
- [ ] determine thumbnail frames across display sizes, scaling modes, and macOS versions;
- [ ] prove participant regions can be transformed correctly into split Mission Control thumbnail regions;
- [ ] prove a click-through overlay can remain visually aligned during Mission Control animation;
- [ ] record failure modes when private data is unavailable;
- [ ] build a compatibility matrix for current supported macOS releases.

**Exit criterion:** a diagnostic tool can print a trustworthy full-screen Space snapshot—including split participant geometry—and a prototype can place one stable decoration on the correct thumbnail region.

## Phase 1 — First fitting

Goal: solve the original single-app full-screen recognition problem **without requiring app adoption**.

- [ ] derive application icon from the actual running bundle;
- [ ] derive localized app title;
- [ ] optionally derive a restrained accent/palette from the app icon;
- [ ] obtain document/window title when available and permitted;
- [ ] application icon overlay;
- [ ] app title overlay;
- [ ] configurable border;
- [ ] configurable tint;
- [ ] click-through rendering;
- [ ] user enable/disable per app;
- [ ] no persistent dependency on native Space identifiers;
- [ ] graceful fallback to app name/icon when richer identity fails.

**Exit criterion:** a user with many dark-mode full-screen apps can identify the desired Space at a glance even though none of those apps know SpaceDress exists.

## Phase 2 — Two-piece suit

Goal: handle split/tiled full-screen as a first-class geometric model.

- [ ] resolve both participants;
- [ ] preserve participant window identity and normalized geometry;
- [ ] derive physical left/right from geometry, never owner-array order;
- [ ] preserve unequal split ratios;
- [ ] render both app icons without ambiguity;
- [ ] define layer clipping/composition rules;
- [ ] test same-app split configurations;
- [ ] test participants from duplicate bundle identifiers/locations.

**Exit criterion:** split Spaces are at least as easy to identify as single-app full-screen Spaces.

## Phase 3 — Desktop Switcher Appearance Specification

Goal: let applications optionally declare preferred switcher appearance without depending on SpaceDress internals.

The public standard is the **Desktop Switcher Appearance Specification (DSAS)**. Its document format is the **Desktop Switcher Appearance Manifest (DSAM)**.

- [ ] finalize the platform-neutral manifest vocabulary;
- [ ] finalize the macOS app-bundle discovery profile;
- [ ] validate manifest schema;
- [ ] bundle-icon source;
- [ ] app-title and document-title sources;
- [ ] border, tint, icon, text, and bitmap image layers;
- [ ] strict local-resource containment;
- [ ] unknown-field/version behavior;
- [ ] renderer/user-policy override rules;
- [ ] manifest diagnostics and validation tool;
- [ ] publish compatibility examples;
- [ ] document how another renderer can implement DSAS without importing SpaceDress configuration semantics.

**Exit criterion:** an unrelated application can add a declarative DSAM and reliably influence a conforming desktop-switcher renderer without referencing SpaceDress.

## Phase 4 — Wardrobe

Goal: polished end-user configuration using **SpaceDress's private multi-app user manifest**.

- [ ] define internal multi-app manifest structure;
- [ ] application selectors keyed primarily by bundle identifier with optional bundle-location/signing refinement;
- [ ] canonical unordered two-app pair selectors for split-specific styling;
- [ ] pair-level container styling and contextual participant overrides;
- [ ] runtime participant/window instance model distinct from process identity;
- [ ] current-instance styling even when no durable document identity exists;
- [ ] durable per-document/per-instance rules only when stable identity exists;
- [ ] reusable per-app instance style sets;
- [ ] manual instance-style selection;
- [ ] sequential instance-style leasing so concurrent same-app windows receive different styles without relying on Mission Control order;
- [ ] defined style-pool overflow behavior that remains visually distinguishable;
- [ ] direct Mission Control context action such as **Customize This Instance** without making the normal overlay input-blocking;
- [ ] direct **Customize This Split Pair** action for split Spaces;
- [ ] visual style editor;
- [ ] per-app rules;
- [ ] reusable user style presets;
- [ ] preview without entering Mission Control;
- [ ] export/import SpaceDress configuration;
- [ ] accessibility controls for contrast, motion, and transparency;
- [ ] privacy control for document titles;
- [ ] keep runtime instance lease/cache state separate from durable user configuration;
- [ ] keep the internal user manifest outside DSAS conformance/versioning.

**Exit criterion:** a user can distinguish and directly customize several visually identical fullscreen windows from the same application, and recurring split pairs can have their own styling without relying on native Space identity or left/right ordering.

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
