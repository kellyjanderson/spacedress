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

The **participant/window is also the runtime instance boundary** for styling. This is intentionally finer grained than application process identity: several fullscreen windows can belong to the same bundle and the same process while still requiring distinct SpaceDress treatment.

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

PID is excellent for mapping current system data to `NSRunningApplication`. It is deliberately useless for durable persistence and may be shared by many visible window instances.

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
6. retain PID and window observations separately for runtime participant disambiguation.

If two processes originate from different app-package paths, they remain distinct even if their bundle identifiers match.

If multiple windows originate from the same exact bundle/process, application identity is shared while participant identity remains distinct.

## Runtime instance identity

For user-facing instance styling, **instance means the current participant/window**.

Conceptually:

```text
ParticipantInstanceObservation
  applicationIdentity
  pid?
  windowID?
  stableDocumentIdentity?
  region?
```

`pid` and `windowID` can make current-session correlation excellent. They do not acquire durability merely because SpaceDress caches them.

A current-instance style may therefore be attached to a reconciled participant lifetime without becoming a persistent identity rule.

When a real stable document/application-specific identity is available, that identity may support a durable instance/document rule. Window title alone does not qualify.

## Appearance and identity are separate

Application identity answers **what app is this?** Participant identity answers **which visible window/instance is this right now?** Appearance answers **how should it be represented?**

The default appearance is derived automatically from the running app. A future application-supplied DSAM may refine that application-side appearance. SpaceDress's multi-app user manifest may then apply application, split-pair, and instance-specific user rules.

None of these appearance layers create Space identity.

## Persistent selectors

User intent is stored against logical identity rather than a Space observation.

### Application selector

The normal durable app selector is progressively specific:

```text
AppSelector
  bundleIdentifier
  bundlePath?          # optional exact-copy disambiguation
  teamIdentifier?      # optional authenticity constraint
```

The bundle identifier is the default persistent app identity. Optional path/signing fields are refinements, not mandatory parts of every app rule.

Examples:

```text
"Dress every com.apple.Safari full-screen participant this way"

"Dress the development build of Example.app at /Applications/Dev/Example.app differently"
```

### Split-pair selector

A recurring pair of applications forms a canonical unordered multiset of application selectors:

```text
ApplicationPairSelector
  members[2]
```

Therefore:

```text
App A + App B == App B + App A
```

and:

```text
App A + App A
```

is a valid same-app pair.

Physical left/right is deliberately excluded from persistent pair identity. Current geometry may be used after matching to apply side-specific presentation.

### Durable document/instance selector

A durable per-instance rule requires a genuinely stable identity such as an explicitly supported document URL or future app-specific persistent identifier.

A title or current window ID may be used for live matching/reconciliation but is not automatically durable identity.

## Instance style slots

SpaceDress may automatically distinguish concurrent windows from the same app by leasing reusable instance styles.

Conceptually:

```text
InstanceStyleSet
  appSelector
  styles[]
  assignmentPolicy

InstanceStyleLease
  participantObservation
  styleSlot
```

A sequential policy assigns the first available slot to each newly observed unmatched participant and holds that lease while the participant remains live/reconciled.

The allocation order is observation/lease state, not Mission Control position. The persistent configuration stores the style set and assignment policy; the current lease map is runtime/session state unless a stable document/instance binding exists.

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

SpaceDress may reconcile snapshots to reduce visual churn and preserve current-instance style leases within a live session.

Signals, strongest first where available:

1. exact participant window ID plus PID continuity;
2. exact set of participant PIDs/window IDs at the Space level;
3. native reference continuity during the same session;
4. participant bundle identities;
5. participant geometry;
6. stable document identity when available;
7. display association;
8. Mission Control position.

Mission Control position is deliberately weak because Spaces may reorder.

Reconciliation creates continuity for rendering and runtime instance assignments. It does **not** convert a session observation into durable user identity.

## Restart and reboot behavior

On SpaceDress restart, current live window IDs may still allow confident reconstruction of current-instance style leases. That is a useful optimization, not a persistence contract.

On application restart or system reboot, SpaceDress reconstructs current observations from scratch and reapplies durable application, pair, and document/instance rules to logical identities it can prove.

Purely current-window assignments may not survive those events. The UI must distinguish that case from durable document binding rather than silently promising persistence.

Native Space IDs may be used as hints for diagnostics/reconciliation, but a mismatch must not orphan durable user styling.

## Regular Desktop Spaces

A regular desktop can have zero, one, or many applications and no privileged "owner." SpaceDress should model it as a participant set rather than inventing one.

Possible future persistent identity for regular desktops may use a user-created logical desktop record plus fuzzy reconciliation, but that problem is explicitly secondary to full-screen Spaces.

See also [ADR 0006 — Layered user style selectors](decisions/0006-layered-user-style-selectors.md).
