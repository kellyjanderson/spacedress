# SpaceDress User Configuration Model

SpaceDress needs a durable way for one user to customize **many applications**. That configuration is product machinery, not an interoperability standard.

This document deliberately separates it from the [Desktop Switcher Appearance Specification](../spec/README.md).

## Boundary

There are two configuration authorities:

1. **application authority** — information derived from the running application, optionally refined by an application-supplied Desktop Switcher Appearance Manifest (DSAM);
2. **user authority** — SpaceDress's internal multi-app manifest and global settings.

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
- does a user override an app's declared color, title, or artwork?
- how are privacy and accessibility preferences applied globally?

Putting both concerns into DSAS would make the public standard carry SpaceDress-specific policy and targeting machinery.

## Internal multi-app manifest

The eventual persisted representation may look conceptually like:

```text
SpaceDressUserManifest
  formatVersion
  globalPreferences
  presets[]
  applicationRules[]
  documentRules[]?
```

An application rule may contain:

```text
ApplicationRule
  selector
    bundleIdentifier
    bundlePath?
    teamIdentifier?
  appearanceOverrides
  enabled
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

## User override

Matching user rules are applied after the application-side appearance is resolved.

Examples:

- replace a hard-to-read icon with a custom local image;
- give all terminal apps one border family;
- assign different accents to two copies of the same application;
- suppress document titles globally;
- force a specific title for a writing project;
- turn off an app-supplied tint while keeping its icon treatment.

## Independence from native Space identity

The user manifest must not persist customization against a native Space ID, UUID, index, or Mission Control position.

Rules target durable logical identity. Native Space identifiers may be cached only as ephemeral reconciliation hints.

## Evolution policy

Because this is internal machinery, SpaceDress may migrate it as the product evolves.

That freedom is intentional. The project should not accidentally create a second public standard merely because the file becomes human-readable.
