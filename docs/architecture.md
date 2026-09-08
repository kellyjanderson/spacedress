# Architecture

SpaceDress is organized around a simple boundary: **macOS tells us what appears to exist; SpaceDress decides what it means and how to dress it.**

The operating-system side is unstable. The project-owned model should not be.

## Dependency direction

```text
┌─────────────────────────────────────────────────────────────┐
│                       Presentation                          │
│ Mission Control observer · Overlay renderer · Settings UI  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────v──────────────────────────────┐
│                       Application                           │
│ Snapshot coordinator · Identity resolver · Style resolver  │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────v──────────────────────────────┐
│                         Domain                              │
│ SpaceSnapshot · Participant · AppIdentity · Style · Rules  │
└──────────────────────────────^──────────────────────────────┘
                               │
┌──────────────────────────────┴──────────────────────────────┐
│                    Platform adapters                        │
│ SkyLight/CGS · Accessibility · NSWorkspace · Bundle · AX   │
└─────────────────────────────────────────────────────────────┘
```

Platform adapters depend on system interfaces and translate them into domain models. Domain code does not call private interfaces.

## Major components

### `SpaceTopologyProvider`

Produces the currently observed Space topology per display.

Responsibilities:

- enumerate Spaces;
- report native identifiers as opaque observation data;
- report current/visible status where known;
- classify likely Space kind when evidence supports it;
- expose source/provenance and confidence when classification is uncertain.

Possible implementations may use private SkyLight/CGS APIs, `com.apple.spaces` preference data for diagnostics/fallbacks, or future supported APIs.

### `SpaceOwnershipProvider`

Determines participant PIDs/windows for a Space.

A full-screen Space normally has one application participant; a tiled/split full-screen Space has two. The provider may combine:

- private Space-owner APIs;
- private managed-space metadata;
- per-Space window enumeration;
- Accessibility information;
- current-space observations.

It must represent ambiguity instead of silently inventing an owner.

### `ParticipantGeometryProvider`

Associates participant windows with their actual screen-space geometry.

The geometry source may combine:

- per-Space window IDs from private SkyLight/SLS discovery;
- public Quartz Window Services window descriptions and bounds;
- private window-bounds calls when necessary;
- Accessibility position/size as a corroborating or fallback source.

The provider stores a **rectangle first** and derives labels such as `left` or `right` second.

For a two-window horizontal split:

```text
smaller participant midX → physical left
larger participant midX  → physical right
```

Unequal width is preserved. Owner-array order, PID order, window-list order, and Mission Control ordering are never treated as placement semantics.

A normalized participant region can be computed relative to the union/content frame of the tiled participants and later transformed into the Mission Control thumbnail frame. This allows a native 70/30 split to remain 70/30 in the overlay.

If geometry is missing or contradictory, placement becomes unknown rather than guessed.

### `ApplicationResolver`

Maps a runtime process to project-owned application identity.

Primary sources should include `NSRunningApplication`, `NSWorkspace`, and `Bundle` metadata:

- PID;
- bundle identifier;
- canonical bundle URL when available;
- localized app name;
- code-signing/team identity when later justified;
- bundle icon.

Two copies of an application with the same bundle identifier but different bundle locations are distinguishable. Two running instances of the same bundle are distinguishable at runtime by PID even if they share an appearance.

### `DocumentResolver`

Optionally extracts document/window identity and display title from Accessibility or application-specific public metadata.

Document identity is opportunistic. A changing window title is not automatically a stable document key.

Document information is privacy-sensitive and should not be collected or logged unless a feature needs it.

### `MissionControlObserver`

Detects Mission Control lifecycle and obtains or infers Space-thumbnail frames.

This is expected to be one of the most macOS-version-sensitive adapters. It should support multiple strategies:

1. Accessibility-derived frames when available;
2. platform-specific layout inference;
3. fail closed when the renderer cannot align safely.

The observer does **not** own style or application identity.

### `DerivedAppearanceProvider`

Builds the useful no-integration baseline from the actual running application.

Potential inputs include:

- real bundle icon;
- localized application name;
- window/document title when allowed;
- deterministic palette information derived from the icon;
- user accessibility/privacy policy.

This path is mandatory product behavior. SpaceDress must remain useful when zero applications implement DSAS.

### `AppearanceManifestLoader`

Discovers and validates an optional **Desktop Switcher Appearance Manifest (DSAM)** supplied by the application under the **Desktop Switcher Appearance Specification (DSAS)**.

The loader begins from the actual running bundle. It never executes application-supplied code.

Missing manifests are normal. Invalid manifests produce diagnostics and leave the derived application appearance usable.

### `UserConfigurationLoader`

Loads SpaceDress's internal multi-app user manifest.

It may contain selectors, rules, overrides, presets, and other SpaceDress-specific state. This format is **not DSAS**, must not be exposed as though it were an industry contract, and may evolve with SpaceDress's needs.

See [`user-configuration-model.md`](user-configuration-model.md).

### `StyleResolver`

Combines the two configuration authorities in deterministic order.

Application-side appearance is built first:

1. derive a baseline from the running app;
2. overlay any valid DSAM declarations supplied by that app.

User intent is then applied:

3. apply matching entries from the SpaceDress multi-app user manifest;
4. enforce global accessibility/privacy/safety policy;
5. produce a resolved appearance.

The user has final authority. Split Spaces resolve each participant independently before composition.

### `OverlayRenderer`

Draws click-through decoration aligned with Mission Control thumbnails.

Properties:

- transparent/borderless overlay surfaces;
- no assumption that the overlay owns Mission Control interaction;
- ignores mouse input unless a future explicit feature requires otherwise;
- tracks Mission Control geometry changes;
- maps normalized split participant regions into the thumbnail;
- honors accessibility settings such as Reduce Transparency and Increase Contrast where practical;
- removes itself immediately when alignment confidence is lost.

## Data flow

```text
observe topology
      │
      v
resolve owners/windows ──> resolve participant geometry
      │                               │
      v                               │
resolve applications ────────┐        │
      │                      │        │
      v                      │        │
derive app appearance       │        │
      │                      │        │
optional DSAM overlay       │        │
      └──────────────┬───────┘        │
                     v                │
               SpaceSnapshot <────────┘
                     │
SpaceDress user      │
manifest ───────────>│
                     v
               resolve styles
                     │
                     v
             ResolvedDecoration
                     │
Mission Control frame┼──────────────┐
                     v              │
               OverlayRenderer <────┘
```

## Capability model

Private APIs fail in ways public APIs usually do not: symbols disappear, data shapes change, permissions alter visibility, and results become incomplete.

SpaceDress should model capabilities explicitly, for example:

```text
spaceTopology        available / degraded / unavailable
spaceOwners          available / partial / unavailable
spaceWindows         available / partial / unavailable
participantGeometry  exact / corroborated / inferred / unavailable
missionControlFrames exact / inferred / unavailable
documentTitles       available / permissionDenied / unsupported
```

A feature asks whether its required capabilities exist. It does not assume that launch success implies platform compatibility.

## Read-only private API policy

Private read interfaces are acceptable when all of the following are true:

- the information materially improves the full-screen use case;
- no supported public interface provides equivalent data;
- the call is isolated behind a narrow adapter;
- absence/failure is handled safely;
- compatibility is probed per macOS release;
- the app does not require SIP weakening.

Private mutation receives a much higher bar and requires an ADR.

## Process and bundle identity

Do not collapse application identity to `bundleIdentifier` alone.

A useful runtime identity can include:

```text
ApplicationObservation
  pid
  bundleIdentifier?
  bundleURL?
  executableURL?
  localizedName?
  launchDate?
```

A persistent selector can choose how specific it needs to be:

```text
ApplicationSelector
  bundleIdentifier
  bundlePath?          # distinguishes copied/dev builds
  teamIdentifier?      # optional strengthening
```

PID is never persisted as application identity.

## Split/tiled Spaces

The domain model stores participants with window and geometry metadata. It must not flatten a split Space into whichever owner happens to be returned first.

A participant should be able to carry:

```text
SpaceParticipant
  application
  windowID?
  screenBounds?
  normalizedRegion?
  physicalSide?       # derived convenience: left/right/etc.
  geometryConfidence
```

Rendering rules:

- show both participant identities by default;
- clip participant decoration to its normalized region when geometry is reliable;
- preserve the native split ratio;
- fall back to a neutral two-icon composition when participant geometry is unknown;
- a participant cannot use its app declaration to cover or impersonate another participant's region.

## Public standard versus SpaceDress configuration

**DSAS is application-facing interoperability.** It describes how an app can express preferred switcher appearance.

**The SpaceDress multi-app user manifest is product configuration.** It describes which apps a particular user wants styled and how.

They are deliberately separate. Shared visual concepts do not make their file formats one standard.

## Failure philosophy

Prefer a missing decoration to a wrong decoration.

Wrong-space overlays destroy trust quickly. If identity or geometry becomes ambiguous during Mission Control animation, SpaceDress should reduce decoration or hide it rather than confidently label the wrong thumbnail.
