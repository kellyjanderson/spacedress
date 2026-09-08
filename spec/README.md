# Desktop Switcher Appearance Specification — Draft 0.1

The **Desktop Switcher Appearance Specification (DSAS)** defines a small declarative format that lets an application express how it would prefer to be represented in a desktop/workspace switcher.

A document conforming to DSAS is a **Desktop Switcher Appearance Manifest (DSAM)**.

DSAS is intentionally **not named after SpaceDress**. SpaceDress is one implementation and the initial steward of the draft. Another renderer should be able to implement the same application-facing contract without adopting SpaceDress branding, user configuration, or internal architecture.

> [!WARNING]
> Version `0.1` is experimental. The shape is published early so real applications and alternate renderers can challenge it before a stable `1.0` contract is declared.

## Scope

DSAS standardizes **application-supplied appearance metadata**.

It does not standardize:

- how a renderer discovers ordinary app metadata when no DSAM exists;
- a user's multi-app rule database;
- SpaceDress's settings or persistence format;
- desktop/Space identity;
- window management;
- switcher layout or thumbnail spacing.

A renderer is expected to remain useful when an application has **no DSAM at all**.

## Design goals

A manifest should be able to request:

- border styling;
- translucent color overlay/tint;
- the application's real bundle icon;
- an app-bundled replacement icon/image;
- application title;
- document/window title;
- literal identifying text;
- local bitmap overlay artwork;
- behavior that remains sensible when one switcher item contains multiple app participants.

It should not become a general extension runtime.

## Terminology

**Specification / DSAS** — this application-facing interoperability contract.  
**Manifest / DSAM** — one application-supplied document conforming to DSAS.  
**Reader** — software that discovers and interprets a DSAM.  
**Renderer** — software that turns a resolved appearance into visible switcher decoration.  
**Participant** — one application/window represented within a switcher item.  
**Application baseline** — appearance a renderer derives without DSAM, such as app icon/name.  
**User configuration** — renderer/product-specific user policy; explicitly outside DSAS.

## Relationship to derived defaults

DSAS does **not** require applications to repeat metadata the platform already exposes.

A renderer may first derive a useful application baseline from platform metadata. On macOS, examples include:

- actual running bundle icon;
- localized application name;
- window/document title when permitted;
- renderer-derived accent colors or other bounded presentation defaults.

If a valid DSAM exists, its declarations refine the application's representation according to the renderer's resolution policy.

A missing DSAM is normal, not an error.

## User authority

DSAS expresses an application's preference, not a command.

A renderer or user may override, suppress, simplify, or ignore application-supplied appearance for accessibility, privacy, policy, safety, consistency, or user preference.

DSAS deliberately does not define the format of those user rules.

## Platform profiles

The core manifest vocabulary is intended to be implementation-neutral. Discovery of a DSAM is platform-specific.

Draft 0.1 defines a **macOS application-bundle profile** because SpaceDress is the initial implementation. Other platform profiles may be added without renaming the standard.

## macOS discovery profile

A reader begins from the **actual bundle URL of the running application**, not a search by application name or bundle identifier.

This matters because two running applications may share a bundle identifier while originating from different package locations.

### Explicit declaration

An app may add the `Info.plist` key:

```xml
<key>DesktopSwitcherAppearanceManifest</key>
<string>DesktopSwitcherAppearance/manifest.json</string>
```

The value is a path **relative to `Contents/Resources`**.

The resolved canonical path must remain inside `Contents/Resources`. Paths that escape through traversal or equivalent indirection are rejected.

### Conventional fallback

If no explicit key is present, readers may look for:

```text
Contents/Resources/DesktopSwitcherAppearance/manifest.json
```

If neither exists, the application simply has no DSAM.

## Manifest root

```json
{
  "$schema": "https://raw.githubusercontent.com/kellyjanderson/spacedress/main/spec/desktop-switcher-appearance.schema.json",
  "manifestVersion": "0.1",
  "name": "Example appearance",
  "layers": []
}
```

### `$schema`

Optional URI for editor/validator tooling.

### `manifestVersion`

Required. Draft 0.1 readers accept exactly `"0.1"`.

While the specification is pre-1.0, a reader encountering an unsupported manifest version should ignore the DSAM and retain its baseline behavior rather than guess compatible semantics.

### `name` and `description`

Optional human-readable metadata for diagnostics/settings.

### `layers`

Required ordered array of visual layers. Earlier entries are conceptually below later entries, subject to renderer safety rules.

A renderer may clamp, suppress, or simplify layers when necessary for accessibility, privacy, multi-participant composition, thumbnail size, or reliable alignment.

## Shared layer fields

Each layer may have:

```json
{
  "id": "optional-stable-name",
  "type": "...",
  "contexts": ["single", "split"]
}
```

`id` is optional and exists for diagnostics/future override targeting. It should be unique within one manifest.

`contexts` is optional. When absent, a layer applies to both single-participant and split/multi-participant contexts. Draft 0.1 values are `single` and `split`.

## Placement

Most content layers use one of:

```text
topLeading   top   topTrailing
leading      center trailing
bottomLeading bottom bottomTrailing
fill
```

Logical `leading`/`trailing` are preferred in the manifest rather than physical left/right. A renderer can map them according to platform/user-interface direction while still using physical window geometry to identify participant regions.

For image layers, `fill` refers to the layer's **permitted participant region**, not necessarily the entire switcher item.

## Semantic sizes

Draft 0.1 uses semantic sizes rather than raw pixels:

```text
small
medium
large
xLarge
```

Renderers choose actual dimensions based on thumbnail size, display scale, accessibility settings, and available participant region.

## Border layer

```json
{
  "type": "border",
  "color": "#7A5CFF",
  "width": "medium",
  "cornerRadius": "system",
  "opacity": 1.0
}
```

In a multi-participant switcher item, an app's border applies to that app's participant region when geometry is reliable. Application-supplied data does not claim the outer border of other participants.

## Tint layer

```json
{
  "type": "tint",
  "color": "#7A5CFF",
  "opacity": 0.14
}
```

Tint is composited over the permitted participant region. Renderers should clamp excessive opacity that would obscure native thumbnail content.

## Icon layer

Use the application's platform icon:

```json
{
  "type": "icon",
  "source": "bundleIcon",
  "placement": "topTrailing",
  "size": "large",
  "background": "material"
}
```

Or a local manifest resource:

```json
{
  "type": "icon",
  "source": "resource",
  "resource": "DesktopSwitcherAppearance/alternate-icon.png",
  "placement": "topTrailing",
  "size": "large"
}
```

Under the macOS profile, `bundleIcon` means the icon of the **actual runtime app bundle**. Readers should use macOS icon services rather than requiring authors to know whether the icon originated as `.icns` or a compiled asset catalog.

## Text layer

```json
{
  "type": "text",
  "source": "appTitle",
  "placement": "bottomLeading",
  "size": "medium",
  "weight": "semibold",
  "background": "material"
}
```

Text sources are `appTitle`, `documentTitle`, and `literal`. A literal layer also supplies `text`.

A renderer may replace `documentTitle` with `appTitle` or hide it when the user has disabled document-title display or required data is unavailable. Text presentation does not establish persistent document identity.

## Image layer

```json
{
  "type": "image",
  "resource": "DesktopSwitcherAppearance/watermark.png",
  "placement": "fill",
  "contentMode": "fit",
  "opacity": 0.18,
  "blendMode": "sourceOver"
}
```

Resources are local to the approved application resource root. Draft 0.1 blend-mode names are `sourceOver`, `multiply`, `screen`, and `overlay`; a renderer may simplify unsupported modes with a diagnostic.

## Resource containment

Every manifest resource must be local and resolve under the platform profile's approved application resource root after canonicalization.

For the macOS profile, that root is the application's `Contents/Resources` directory.

Absolute paths, remote URLs, traversal outside the resource root, and resources that require helper execution are rejected.

Resources are decoded as bounded data, not treated as extensions to the application.

## Multi-participant semantics

A DSAM styles **its application participant**, not unrelated participants represented in the same switcher item.

When participant geometry is known:

- tint/image/border layers are clipped to that participant region;
- icon/text placement is relative to that region;
- one app cannot cover another participant.

When geometry is unknown but multiple applications are identified:

- readers/renderers should preserve each identity;
- full-region app tint/image layers may be suppressed;
- icon/text layers may be placed into a neutral multi-participant layout;
- a renderer must not guess ownership geometry from undocumented array order.

DSAS does not standardize how a renderer discovers participant geometry.

## What DSAS does not standardize about users

A product may maintain a multi-app user manifest, rule database, style presets, per-document exceptions, or other configuration.

Those are renderer concerns.

In particular, **SpaceDress's multi-app user manifest is not a DSAM and is not part of DSAS**. It may reuse visual concepts internally without creating a compatibility promise.

## Rendering safety

A conforming renderer may override requested presentation to protect usability:

- clamp opacity or border prominence;
- simplify crowded multi-participant layouts;
- hide document titles for privacy;
- respect accessibility settings;
- suppress a layer when its local resource fails validation;
- suppress all decoration when switcher alignment is unreliable.

The fallback for a bad manifest is a usable baseline, not a broken switcher item.

## Extension policy

The root may contain an optional `extensions` object for namespaced experimental data. Core readers ignore unrecognized namespaces. Extensions do not bypass resource, privacy, or rendering-safety rules.

A broadly useful extension should graduate into a future DSAS version rather than become one renderer's private de facto API.

## Schema and examples

- [`desktop-switcher-appearance.schema.json`](desktop-switcher-appearance.schema.json)
- [`examples/minimal.json`](examples/minimal.json)
- [`examples/author-branded.json`](examples/author-branded.json)
- [`examples/split-friendly.json`](examples/split-friendly.json)

## Version history

See [`CHANGELOG.md`](CHANGELOG.md).
