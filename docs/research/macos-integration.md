# macOS integration research

> Status: working research baseline, not a public Apple compatibility guarantee.

Apple does not expose a supported API for decorating Mission Control's Space thumbnails. SpaceDress therefore needs a layered integration strategy that uses supported APIs wherever possible and isolates undocumented mechanisms where they are unavoidable.

## Capability ladder

### Tier A — supported public APIs

Prefer these for application identity and normal app behavior.

Examples:

- `NSWorkspace` / `NSRunningApplication` for running-process and app-bundle metadata;
- `Bundle` / `Info.plist` for app metadata and SpaceDress manifest discovery;
- AppKit/SwiftUI for SpaceDress windows and settings;
- Accessibility (`AXUIElement`) for permitted UI/window metadata;
- workspace active-Space notifications where useful.

Public does not mean permission-free: Accessibility data for other apps requires user consent.

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
- per-Space window enumeration/capture.

This tier is likely necessary for high-quality full-screen ownership resolution.

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

Private managed-space APIs and the `com.apple.spaces` domain both demonstrate that Space topology is observable. The exact source of truth and current-vs-other-Space behavior must be tested on each supported macOS release.

### Resolve full-screen owners

Historical CGS headers expose `CGSSpaceCopyOwners`, described as returning application PIDs that own a given Space. Other modern SkyLight data/window enumeration paths can provide additional evidence.

This aligns well with the full-screen-first model: owner PID can lead directly to `NSRunningApplication`, and from there to the **actual app package that launched the process**.

Do not assume one owner for split/tiled full-screen. The provider must preserve all participants.

### Resolve app package and icon

Given a PID, `NSRunningApplication` can expose bundle identifier, localized name, and bundle URL when available. The bundle URL is critical for distinguishing separate installed/development copies with the same bundle identifier.

For the icon, prefer the system's icon resolution for that exact bundle (`NSWorkspace`/bundle icon APIs) rather than manually guessing which `.icns` or compiled asset catalog entry represents the app.

The app package can then be inspected for a SpaceDress manifest under the published discovery rules.

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

Do not assume it. Modern reports show some window-to-Space calls behaving differently for the active Space versus other Spaces.

### Mission Control Accessibility hierarchy is stable

Do not assume it. Names, roles, geometry, localization, and exposure have varied.

## Full-screen identity experiment matrix

Before implementation commits to a provider, test at least:

| Case | Expected participant model |
| --- | --- |
| one app, one full-screen window | one app participant |
| same app, second full-screen window | same app identity, distinct runtime/window context |
| two different app copies with same bundle ID | distinct bundle URLs |
| two instances from same exact bundle | distinct PIDs, shared default app style |
| split two different apps | two participants |
| split two windows of same app | two participants sharing app identity |
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

Useful open-source references include:

- Hammerspoon `hs.spaces` — managed Spaces and Mission Control AX inspection;
- NUIKit `CGSInternal` — historical private CGS declarations including Space owners;
- AltTab — mature macOS window/Space integration;
- SpaceNamer / DesktopRenamer / Rename Spaces — Space naming and overlay approaches;
- Control Room — modern private SkyLight per-Space enumeration/capture;
- CloseUp — Mission Control augmentation through overlays;
- Lattice — Space navigation/thumbnail experiments with SIP enabled.

See [`prior-art.md`](prior-art.md) for links and project-specific notes.
