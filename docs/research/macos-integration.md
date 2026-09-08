# macOS integration research

> Status: working research baseline, not a public Apple compatibility guarantee.

Apple does not expose a supported API for decorating Mission Control's Space thumbnails. SpaceDress therefore needs a layered integration strategy that uses supported APIs wherever possible and isolates undocumented mechanisms where they are unavoidable.

## Capability ladder

### Tier A — supported public APIs

Prefer these for application identity, window geometry, and normal app behavior where they provide enough information.

Examples:

- `NSWorkspace` / `NSRunningApplication` for running-process and app-bundle metadata;
- `Bundle` / `Info.plist` for app metadata and optional DSAM discovery;
- Quartz Window Services (`CGWindowListCopyWindowInfo`, window descriptions, `kCGWindowBounds`) for window metadata and screen-space bounds;
- AppKit/SwiftUI for SpaceDress windows and settings;
- Accessibility (`AXUIElement`) for permitted UI/window metadata;
- workspace active-Space notifications where useful.

Public does not mean permission-free: some information about other apps may require Accessibility or Screen Recording consent depending on API and macOS release.

### Tier B — readable system state with undocumented semantics

The `com.apple.spaces` preferences domain has historically exposed managed display/Space topology including fields such as display identifiers, managed-space IDs, types, and UUID-like values.

It is useful for:

- diagnostics;
- cross-checking other providers;
- compatibility research;
- fallback experiments.

It is not treated as a durable schema merely because it is a plist.

### Tier C — private SkyLight/CGS/SLS read SPI

Open-source macOS tools demonstrate private symbols that can provide richer Space/window information, including variants of:

- managed-display Space enumeration;
- all-Space enumeration;
- Space owner PIDs;
- Space membership for windows;
- per-Space window enumeration;
- direct window bounds;
- managed display association.

Current open-source declarations in yabai include `SLSCopyManagedDisplaySpaces`, `SLSCopySpacesForWindows`, `SLSCopyWindowsWithOptionsAndTags`, `SLSGetWindowBounds`, and managed-display helpers. These are private interfaces and therefore evidence, not contracts.

This tier is likely necessary for high-quality full-screen ownership and non-current-Space resolution.

Project policy:

- read-only first;
- dynamically isolate symbols/signatures;
- probe availability at runtime;
- sanitize returned structures immediately into project models;
- maintain macOS-version compatibility tests;
- never assume App Store eligibility;
- SIP remains enabled.

### Tier D — invasive mutation/injection

Examples include injecting into the Dock, modifying Mission Control implementation code, or asking the user to weaken SIP.

This is outside the normal SpaceDress architecture.

A visually equivalent overlay is strongly preferred over modifying Apple's renderer.

## What appears feasible

### List Spaces

Private managed-space APIs and the `com.apple.spaces` domain both demonstrate that Space topology is observable. Contemporary open-source research reports `SLSCopyManagedDisplaySpaces` returning per-display Space data including IDs, UUID-like values, and Space type; full-screen/tiled Spaces are commonly observed as type `4`.

The exact source of truth and current-vs-other-Space behavior must be tested on each supported macOS release.

### Resolve full-screen owners

Historical CGS headers expose `CGSSpaceCopyOwners`, described as returning application PIDs that own a given Space. Other modern SkyLight data/window enumeration paths can provide additional evidence.

This aligns well with the full-screen-first model: owner PID can lead directly to `NSRunningApplication`, and from there to the **actual app package that launched the process**.

Do not assume one owner for split/tiled full-screen. The provider must preserve all participants.

Do not interpret owner-array order as left/right placement.

### Resolve windows belonging to a Space

Private interfaces can correlate window IDs and Space IDs in both directions. A robust probe should test both strategies where available:

- Space → window IDs;
- candidate window IDs → Space IDs.

Window IDs should then be correlated with owner PID/application identity and public/private window metadata.

### Determine split left/right and ratio

**Working hypothesis: this should be reliably derivable from participant window geometry without a dedicated left/right API.**

Apple's public Quartz Window Services documentation states that window descriptions include bounds and that `kCGWindowBounds` is expressed in screen coordinates with the origin at the upper-left of the main display. Current yabai sources also use private `SLSGetWindowBounds` to obtain `CGRect` geometry for window IDs.

For a native two-window horizontal split:

1. identify the two participant window IDs belonging to the tiled full-screen Space;
2. obtain each window's screen-space bounds;
3. map each window to its owner PID/application;
4. compare participant `midX` values;
5. lower `midX` is physical left; higher `midX` is physical right;
6. retain the rectangles rather than throwing them away after assigning side labels.

The union of the participant rectangles can serve as a content frame for normalization:

```text
normalizedX = (window.minX - content.minX) / content.width
normalizedY = (window.minY - content.minY) / content.height
normalizedW = window.width / content.width
normalizedH = window.height / content.height
```

When Mission Control exposes or SpaceDress infers the thumbnail frame, those normalized rectangles can be transformed into thumbnail-local participant regions.

This preserves unequal native splits such as 70/30.

#### What still needs proof

Phase 0 must verify on current supported macOS builds that native tiled/full-screen participant windows expose distinct expected bounds when:

- the split is 50/50;
- the divider is moved to an unequal ratio;
- left and right participants are swapped;
- both participants belong to the same app;
- Mission Control is open;
- the Space is not currently active;
- multiple displays and scaling modes are involved.

If public Quartz bounds are hidden, normalized, or stale in a relevant state, `SLSGetWindowBounds` and Accessibility position/size are fallback/corroborating sources.

The architecture therefore treats geometry as a capability with confidence, not as an unconditional guarantee.

### Resolve app package and icon

Given a PID, `NSRunningApplication` can expose bundle identifier, localized name, and bundle URL when available. The bundle URL is critical for distinguishing separate installed/development copies with the same bundle identifier.

For the icon, prefer the system's icon resolution for that exact bundle (`NSWorkspace`/bundle icon APIs) rather than manually guessing which `.icns` or compiled asset catalog entry represents the app.

The app package can also be inspected for an optional **Desktop Switcher Appearance Manifest (DSAM)** under the DSAS macOS discovery profile. Missing DSAM data is expected and does not weaken the default experience.

### Derive application appearance without integration

SpaceDress's initial product cannot depend on app adoption.

A useful baseline should be derived automatically from ordinary runtime/bundle metadata, including at minimum:

- actual bundle icon;
- localized app name;
- optional document/window title;
- safe generic treatment such as border/text background;
- optionally, a restrained deterministic accent derived from the icon.

This derivation is SpaceDress behavior, not part of DSAS.

### Obtain document/window title

Accessibility can expose window/document attributes for many apps, but support varies and requires permission. Treat this as optional metadata and privacy-sensitive.

### Detect Mission Control and locate thumbnails

Mission Control is owned by the Dock/system UI. Accessibility trees can expose useful Mission Control elements on some releases/configurations, but existing projects report missing/inconsistent labels or frames and sometimes fall back to geometric inference.

SpaceDress should therefore support a strategy chain rather than one hard-coded AX path.

### Overlay the thumbnail

Existing utilities demonstrate transparent overlays that visually augment Mission Control without changing native Space metadata or disabling SIP.

This is the preferred rendering strategy.

## What is intentionally not assumed

### Native Space IDs are stable

Do not assume it.

Some tools report UUIDs persisting across reboots and use them successfully for desktop naming. Other fields are clearly session-oriented, and Apple provides no public persistence contract for the overall structure.

SpaceDress gains little by betting user configuration on undocumented persistence when logical app identity is a better fit for full-screen Spaces anyway.

### `com.apple.spaces` is always current

Do not assume it. Some projects use it successfully; others report stale data and prefer SkyLight. Keep it as one evidence source.

### One private symbol works for every Space

Do not assume it. Window-to-Space calls may behave differently for the active Space versus other Spaces or across macOS releases.

### Owner-array order encodes split side

Do not assume it. Placement is a geometric property and should be established from participant rectangles.

### Mission Control Accessibility hierarchy is stable

Do not assume it. Names, roles, geometry, localization, and exposure have varied.

## Full-screen identity experiment matrix

Before implementation commits to a provider, test at least:

| Case | Expected participant model |
| --- | --- |
| one app, one full-screen window | one app participant |
| same app, second full-screen window | same app identity, distinct runtime/window context |
| two different app copies with same bundle ID | distinct bundle URLs |
| two instances from same exact bundle | distinct PIDs, shared derived app appearance |
| split two different apps 50/50 | two participants with distinct left/right regions |
| split two different apps 70/30 | two participants with preserved unequal normalized regions |
| swap split participants | side follows geometry, not owner ordering |
| split two windows of same app | two participants sharing app identity but distinct window geometry |
| move full-screen Space between displays | identity preserved through re-observation |
| restart SpaceDress | rules re-resolve without relying on native ID |
| reboot macOS / restore apps | rules re-resolve from logical identity |

## Mission Control settings that affect tests

Always record:

- "Displays have separate Spaces";
- "Automatically rearrange Spaces based on most recent use";
- number and arrangement of displays;
- display scaling;
- Reduce Motion/Transparency when testing overlays.

## Research references

Useful primary/open-source references include:

- Apple Quartz Window Services — public window metadata and screen-space bounds;
- yabai — current private SLS declarations and real-world window/Space correlation;
- Hammerspoon `hs.spaces` — managed Spaces and Mission Control AX inspection;
- NUIKit `CGSInternal` — historical private CGS declarations including Space owners;
- AltTab — mature macOS window/Space integration;
- SpaceNamer / DesktopRenamer / Rename Spaces — Space naming and overlay approaches;
- Control Room — modern private SkyLight per-Space enumeration/capture;
- CloseUp — Mission Control augmentation through overlays;
- Lattice — Space navigation/thumbnail experiments with SIP enabled.

See [`prior-art.md`](prior-art.md) for links and project-specific notes.
