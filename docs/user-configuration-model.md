# SpaceDress User Configuration Model

SpaceDress needs a durable way for one user to customize **many applications, recurring split pairs, and individual fullscreen participants**. That configuration is product machinery, not an interoperability standard.

This document deliberately separates it from the [Desktop Switcher Appearance Specification](../spec/README.md).

## Boundary

There are two configuration authorities:

1. **application authority** — information derived from the running application, optionally refined by an application-supplied Desktop Switcher Appearance Manifest (DSAM);
2. **user authority** — SpaceDress's internal multi-app manifest, pair rules, instance assignments, presets, and global settings.

The user authority wins.

## Why this is not DSAS

An application-facing standard and an end-user rule database solve different problems.

DSAS needs to answer questions such as:

- what appearance may this application request for itself?
- how are visual resources bounded?
- what does a renderer do with unknown fields or versions?
- how does an app-supplied layer behave in a split participant region?

SpaceDress user configuration needs to answer questions such as:

- which installed/running app does this rule target?
- should one copied development build look different from another copy?
- should multiple apps share a preset?
- should a particular two-app split have a special treatment?
- how should five concurrent windows of the same app be visually separated?
- should one specific document/window receive its own style?
- does a user override an app's declared color, title, or artwork?
- how are privacy and accessibility preferences applied globally?

Putting those concerns into DSAS would make the public standard carry SpaceDress-specific targeting and persistence machinery.

## Selector hierarchy

Customization becomes more specific as it moves down this hierarchy:

```text
application
  -> split-pair context
  -> instance/document/instance-slot
```

A more specific user selection may override a less specific user selection. Required privacy, accessibility, and rendering-safety policy remains final.

## Application identity

The default durable application key is its bundle identifier.

Conceptually:

```text
ApplicationSelector
  bundleIdentifier
  bundlePath?        # optional exact-copy distinction
  teamIdentifier?    # optional strengthening/disambiguation
```

For normal use, `bundleIdentifier` is enough. The optional fields exist because macOS can run multiple copied/development builds that share a bundle identifier.

If an app has no usable bundle identifier, SpaceDress may use a weaker fallback selector based on its actual bundle/executable location. That should be treated as less portable than a normal bundle-ID rule.

## Split-pair identity

Two application selectors can form a persistent split-pair selector.

```text
ApplicationPairSelector
  members[2]
```

The pair is a **canonical unordered multiset**. It is not keyed by physical left/right placement.

Therefore:

```text
Chrome + VS Code == VS Code + Chrome
```

and a same-app pair is valid:

```text
Chrome + Chrome
```

A pair rule may contain:

- whole-Space/container decoration such as a shared outer border or title;
- contextual participant overrides;
- optional physical-side treatment applied only after current window geometry is known.

Swapping the two windows left/right must not stop the logical pair rule from matching.

## Instance identity

For SpaceDress, an **instance** is a visible Space participant/window, not merely a process.

That distinction matters because several fullscreen Chrome windows may share one bundle identifier and one application process. Several TextEdit documents may likewise be windows of one app process.

Runtime correlation may include:

```text
ParticipantObservation
  application
  pid?
  windowID?
  documentIdentity?
  region?
```

`pid` and `windowID` are useful while they are true, but neither is a durable user identifier.

### Durable document/instance binding

When SpaceDress has a genuinely stable identity, a user may choose to remember styling against it.

Possible stable evidence includes:

- a stable document URL;
- a future application-specific persistent identifier;
- another identity source with an explicit persistence contract.

A window title is not automatically a durable identity. Titles may still be used as explicit matching criteria if the user chooses a title/pattern rule, but that is matching policy rather than identity.

### Current-instance binding

When no stable identity exists, SpaceDress still needs useful instance styling.

A user may assign a style to **this current window/participant**. SpaceDress may preserve the assignment with current window/process observations and a session cache while that instance remains live or can be confidently reconciled.

The UI must not promise persistence across arbitrary application restart or system reboot unless durable evidence exists.

## Instance style sets

An app may have several reusable styles specifically for disambiguating concurrent instances.

Conceptually:

```text
InstanceStyleSet
  appSelector
  styles[]
  assignmentPolicy
```

Example:

```text
Chrome
  style 1: red border
  style 2: blue border
  style 3: green border
  style 4: yellow border
  style 5: purple border
  assignment: sequential
```

Assignment policies initially should include:

- **manual** — instances share the app default until the user chooses a style;
- **sequential** — each newly observed unmatched concurrent instance receives the first available style slot.

Sequential assignment is a **lease**, not persistent identity. The lease remains attached to the reconciled participant while that participant exists. It must not be derived from Mission Control order.

Until the style pool is exhausted, concurrent instances should not receive the same visual slot. Overflow behavior should remain distinguishable, for example by adding an ordinal badge or a deterministic fallback variation rather than silently producing identical thumbnails.

A current assignment cache is operational state, not the durable user manifest. Durable configuration stores the style set and assignment policy; durable instance bindings are stored only when stable identity exists.

## Internal multi-app manifest

The eventual persisted representation may look conceptually like:

```text
SpaceDressUserManifest
  formatVersion
  globalPreferences
  presets[]
  applicationRules[]
  pairRules[]
  durableInstanceRules[]
```

An application rule may contain:

```text
ApplicationRule
  selector
    bundleIdentifier
    bundlePath?
    teamIdentifier?
  appearanceOverrides
  instanceStyleSet?
  enabled
```

A pair rule may contain:

```text
PairRule
  selector
    members[2]
  containerAppearance?
  participantOverrides?
  enabled
```

A durable instance rule may contain:

```text
InstanceRule
  appSelector
  stableDocumentOrInstanceSelector
  appearanceOrPreset
```

This is a conceptual model, **not a frozen JSON schema**.

Until implementation requirements are known, the project should avoid promising field names, file locations, or long-term compatibility for this internal format.

## Derived defaults

No user rule is required for an app to look useful.

The baseline comes from the actual running application. Candidate values include:

- bundle icon;
- localized application name;
- document/window title when allowed;
- a restrained deterministic accent derived from the icon;
- generic safe defaults for border/tint/text treatment.

The exact derivation algorithm is SpaceDress behavior, not DSAS.

## Optional application declaration

If the running app supplies a valid DSAM, its declarations overlay the derived application baseline for fields/layers it explicitly controls.

A missing DSAM is the expected case during initial SpaceDress adoption and is not diagnostically interesting.

## User override precedence

Participant appearance resolves conceptually as:

```text
platform-derived baseline
  -> optional DSAM
  -> application rule
  -> split-pair participant/context rule
  -> explicit durable/current-instance or instance-slot assignment
  -> required global privacy/accessibility/safety policy
```

Whole-pair/container appearance is composed separately so it cannot erase participant identity.

An explicit instance choice is more specific than a generic app or pair-member rule.

Examples:

- replace a hard-to-read app icon with a custom local image;
- give all terminal apps one border family;
- assign different accents to two copies of the same application;
- style Chrome + VS Code differently from either application alone;
- automatically distribute five Chrome windows across five style slots;
- manually choose a unique style for one current browser window;
- remember a style for one document when a stable document identity is available;
- suppress document titles globally;
- turn off an app-supplied tint while keeping its icon treatment.

## Direct Mission Control customization

The intended interaction is to customize the thing the user is currently looking at.

A context action on a fullscreen thumbnail/participant should eventually offer commands such as:

```text
Customize This Instance…
Apply Instance Style >
Customize App Defaults…
Customize This Split Pair…
Auto-assign Instance Styles >
```

The normal SpaceDress visual overlay remains click-through. The exact right-click/context-menu mechanism therefore needs a dedicated integration experiment; adding customization must not turn the entire overlay into an input-blocking replacement for Mission Control.

When a stable document identity exists, the UI may additionally offer a clearly separate **Remember for this document** action. Otherwise the UI should describe the assignment as current-window/current-instance styling.

## Independence from native Space identity

The user manifest must not persist customization against a native Space ID, UUID, index, Mission Control position, PID, or transient window ID.

Rules target logical identity. Native/runtime identifiers may be cached only as ephemeral reconciliation hints.

## Evolution policy

Because this is internal machinery, SpaceDress may migrate it as the product evolves.

That freedom is intentional. The project should not accidentally create a second public standard merely because the file becomes human-readable.

See also [ADR 0006 — Layered user style selectors](decisions/0006-layered-user-style-selectors.md).
