# The Space model

The central modeling rule is simple:

> **A macOS Space identifier identifies an observation of a Space. It does not identify the user's enduring intent.**

This remains true even when a particular identifier appears stable across reboots on a particular macOS release.

## Why this matters

macOS exposes several pieces of Space-related state through undocumented preference data and private SkyLight/CGS interfaces, including integer managed-space IDs, UUID-like values, ordering, display associations, window membership, and full-screen/tile metadata.

Some of those values may persist for long periods. None are part of a public compatibility contract suitable for the foundation of user configuration.

SpaceDress can use them aggressively **inside a snapshot** and cautiously **inside a cache**. It must not make them the durable key for style rules.

## Runtime model

A conceptual snapshot:

```text
SpaceSnapshot
  observationID           # SpaceDress session-local ID
  nativeReferences[]      # opaque native IDs/UUIDs with provenance
  display
  kind                    # fullscreenSingle / fullscreenSplit / desktop / unknown
  participants[]
  missionControlPosition?
  thumbnailFrame?
  confidence
  observedAt
```

`observationID` is created by SpaceDress and exists only to correlate data during a running session.

### Native references

Native references are tagged, not normalized into one magic ID:

```text
NativeSpaceReference
  source                  # SLS, CGS, com.apple.spaces, AX, ...
  kind                    # managedID, uuid, id64, index, ...
  value
  observedAt
```

This makes it possible to compare evidence without pretending two similarly named fields have identical semantics.

## Participant model

```text
SpaceParticipant
  process
  application
  windowID?
  windows[]
  document?
  screenBounds?
  normalizedRegion?
  physicalSide?
  geometryConfidence
```

A single full-screen Space usually has one participant. A split/tiled Space has two first-class participants.

The **rectangle is primary**. `physicalSide` is a derived convenience such as `left` or `right`, never the source of truth.

### Split geometry

For a conventional horizontal two-window split, the participant with the lower `midX` is physically left and the higher `midX` is physically right.

SpaceDress retains the actual bounds so it can preserve unequal ratios and future arrangements instead of collapsing the model into two labels.

Normalize participant rectangles relative to the tiled content frame (normally the union of participant bounds):

```text
ParticipantRegion
  x       # 0...1
  y       # 0...1
  width   # 0...1
  height  # 0...1
```

The normalized region can later be mapped into the Mission Control thumbnail frame.

Owner-array order, PID order, window enumeration order, and Mission Control position do not define participant side.

### Process observation

```text
ProcessObservation
  pid
  executableURL?
  launchDate?
```

PID is excellent for mapping current system data to `NSRunningApplication`. It is deliberately useless for persistence.

### Application identity

At runtime:

```text
ApplicationIdentity
  bundleIdentifier?
  canonicalBundleURL?
  localizedName
  teamIdentifier?
```

The `canonicalBundleURL` matters because macOS can run multiple copies/builds of an application that share a bundle identifier.

Resolution strategy:

1. start from owner PID/window when available;
2. obtain `NSRunningApplication` for that PID;
3. prefer its actual `bundleURL` rather than searching `/Applications` by name;
4. construct `Bundle(url:)` for metadata;
5. use `NSWorkspace.shared.icon(forFile:)` or equivalent system icon resolution for the app's current bundle;
6. retain PID separately for same-bundle runtime disambiguation.

If two processes originate from different app-package paths, they remain distinct even if their bundle identifiers match.

If two processes originate from the exact same package, their runtime PIDs distinguish them, while derived application appearance normally remains shared.

## Appearance and identity are separate

Application identity answers **what app/window is this?** Appearance answers **how should it be represented?**

The default appearance is derived automatically from the running app. A future application-supplied DSAM may refine that application-side appearance. SpaceDress's multi-app user manifest may override it.

None of these appearance layers create Space identity.

## Persistent selectors

User intent is stored as a rule against logical identity rather than a Space observation.

Examples:

```text
"Dress every com.apple.Safari full-screen Space this way"

"Dress the development build of Example.app at /Applications/Dev/Example.app differently"

"Use this style when App A and App B form a split Space"
```

A persistent app selector can therefore be progressively specific:

```text
AppSelector
  bundleIdentifier
  bundlePath?          # optional exact-copy disambiguation
  teamIdentifier?      # optional authenticity constraint
```

A split selector may consider the participant set and geometry when that distinction is meaningful, but never persists a native Space ID as identity.

## Titles

A title is presentation data, not identity by default.

Potential title sources:

- app localized name;
- application-provided literal label through DSAM;
- Accessibility window title;
- Accessibility document title/path;
- user-supplied label.

Window titles often contain volatile state such as unsaved markers, tab names, playback position, or search text. Do not use them as persistent document identity merely because they are available.

When a stable document URL is exposed and the user chooses document-specific rules, it can become part of a selector. Such paths are private data and should remain local.

## Reconciliation across observations

SpaceDress may reconcile snapshots to reduce visual churn within a session.

Signals, strongest first where available:

1. exact set of participant PIDs/window IDs;
2. native reference continuity during the same session;
3. participant bundle identities;
4. participant geometry;
5. display association;
6. Mission Control position.

Mission Control position is deliberately weak because Spaces may reorder.

Reconciliation creates continuity for rendering. It does **not** convert a session observation into durable user identity.

## Restart and reboot behavior

On app restart or system reboot, SpaceDress reconstructs current observations from scratch and reapplies persistent rules to the logical applications/documents it finds.

It may use previously observed native IDs as hints for diagnostics, but a mismatch must not orphan a user's style.

This is the intended behavior even if future testing shows that a particular UUID is stable in most cases. A more stable native field is useful optimization data, not a reason to weaken the model.

## Regular Desktop Spaces

A regular desktop can have zero, one, or many applications and no privileged "owner." SpaceDress should model it as a participant set rather than inventing one.

Possible future persistent identity for regular desktops may use a user-created logical desktop record plus fuzzy reconciliation, but that problem is explicitly secondary to full-screen Spaces.
