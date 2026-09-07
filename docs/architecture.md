# Architecture

SpaceDress is organized around a simple boundary: **macOS tells us what appears to exist; SpaceDress decides what it means and how to dress it.**

The operating system side is unstable. The project-owned model should not be.

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

### `ApplicationResolver`

Maps a runtime process to project-owned application identity.

Primary sources should include `NSRunningApplication`, `NSWorkspace`, and `Bundle` metadata:

- PID;
- bundle identifier;
- canonical bundle URL when available;
- localized app name;
- code-signing/team identity when later justified;
- bundle icon.

Two copies of an application with the same bundle identifier but different bundle locations are distinguishable. Two running instances of the same bundle are distinguishable at runtime by PID even if they share a style.

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

### `StyleManifestLoader`

Discovers and validates app-bundled SpaceDress manifests. It never executes application-supplied code.

Invalid manifests produce diagnostics and fall back to synthesized defaults.

### `StyleResolver`

Combines styling sources in deterministic precedence:

1. explicit user rule/override;
2. selected user style package;
3. valid app-bundled manifest;
4. synthesized application style;
5. neutral fallback.

Split Spaces resolve each participant independently before composition.

### `OverlayRenderer`

Draws click-through decoration aligned with Mission Control thumbnails.

Properties:

- transparent/borderless overlay surfaces;
- no assumption that the overlay owns Mission Control interaction;
- ignores mouse input unless a future explicit feature requires otherwise;
- tracks Mission Control geometry changes;
- honors accessibility settings such as Reduce Transparency and Increase Contrast where practical;
- removes itself immediately when alignment confidence is lost.

## Data flow

```text
observe topology
      │
      v
resolve owners ───────> resolve apps ───────> optional document metadata
      │                       │
      └──────────────┬────────┘
                     v
               SpaceSnapshot
                     │
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

The domain model stores participants as a collection with optional geometry/order metadata. It must not flatten a split Space into whichever owner happens to be returned first.

Rendering rules:

- show both participant identities by default;
- clip app-owned decoration to that participant's region when reliable geometry exists;
- fall back to a neutral two-icon composition when participant geometry is unknown;
- a participant cannot use its app manifest to cover or impersonate the other participant's region.

## Failure philosophy

Prefer a missing decoration to a wrong decoration.

Wrong-space overlays destroy trust quickly. If identity or geometry becomes ambiguous during Mission Control animation, SpaceDress should reduce decoration or hide it rather than confidently label the wrong thumbnail.
