# SpaceDress Style Manifest — v0.1

The SpaceDress Style Manifest is a small declarative standard that lets a macOS application describe how it would prefer to be identified when its content occupies a Mission Control Space thumbnail.

The standard is intentionally independent of the SpaceDress app implementation. Another renderer may implement it if it follows the same discovery, security, and layer semantics.

> [!WARNING]
> Version `0.1` is experimental. The project is publishing the shape early so real applications can challenge it before a stable `1.0` contract is declared.

## Goals

A manifest should be able to request:

- border styling;
- translucent color overlay/tint;
- the application's real bundle icon;
- an app-bundled replacement icon/image;
- application title;
- document/window title;
- literal identifying text;
- local bitmap overlay artwork;
- behavior that remains sensible in split/tiled full-screen Spaces.

It should not become a general extension runtime.

## Non-goals for v0.1

The base standard does not support dynamic behavior, remote resources, animation/video, interactive controls, or Mission Control layout changes. Styling is data interpreted by the renderer.

## Terminology

**Reader** — software that discovers and interprets a SpaceDress manifest.  
**Renderer** — software that turns a resolved manifest into visible decoration.  
**Participant** — one application/window participant in a full-screen Space.  
**App manifest** — a manifest shipped inside a `.app` bundle.  
**User style** — a user-selected style package using the same visual layer vocabulary.

## Discovery inside an app bundle

A reader begins from the **actual bundle URL of the running application**, not a search by application name.

This is important because two running applications may share a bundle identifier while originating from different package locations.

### Explicit declaration

An app may add the custom `Info.plist` key:

```xml
<key>SpaceDressStyleManifest</key>
<string>SpaceDress/manifest.json</string>
```

The value is a path **relative to `Contents/Resources`**.

The resolved canonical path must remain inside `Contents/Resources`. Paths that escape through traversal or equivalent indirection are rejected.

### Conventional fallback

If no `SpaceDressStyleManifest` key is present, readers should look for:

```text
Contents/Resources/SpaceDress/manifest.json
```

If neither exists, the app has no bundled SpaceDress manifest and the renderer may synthesize a default from normal application metadata.

## Resolution precedence

The SpaceDress application resolves style sources in this order:

1. explicit user override/rule;
2. user-selected style package;
3. valid app-bundled manifest;
4. synthesized style from actual bundle icon/name;
5. neutral fallback.

App authors cannot override user policy. A user may globally suppress document titles, bitmap overlays, tints, or all app-supplied styling.

## Manifest root

```json
{
  "$schema": "https://raw.githubusercontent.com/kellyjanderson/spacedress/main/spec/spacedress-style.schema.json",
  "manifestVersion": "0.1",
  "name": "Example style",
  "layers": []
}
```

### `$schema`

Optional URI for editor/validator tooling.

### `manifestVersion`

Required. v0.1 readers accept exactly `"0.1"`.

While the standard is pre-1.0, a reader encountering an unsupported manifest version should ignore the app manifest and fall back rather than guess compatible semantics.

### `name` and `description`

Optional human-readable metadata for diagnostics/settings.

### `layers`

Required ordered array of visual layers. Earlier entries are conceptually below later entries, subject to renderer safety rules.

A renderer may clamp, suppress, or simplify layers when necessary for accessibility, privacy, split composition, thumbnail size, or reliable Mission Control alignment.

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

`contexts` is optional. When absent, a layer applies to both single and split full-screen contexts. Allowed values are `single` and `split`.

## Placement

Most content layers use one of:

```text
topLeading   top   topTrailing
leading      center trailing
bottomLeading bottom bottomTrailing
fill
```

`leading`/`trailing` are used instead of left/right. For image layers, `fill` refers to the layer's permitted region, not necessarily the entire Mission Control thumbnail.

## Semantic sizes

v0.1 uses semantic sizes rather than raw pixels:

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

In split full-screen, an app manifest's border applies to that app's participant region when geometry is reliable. Only user-level whole-Space styling may assume ownership of the entire outer border.

## Tint layer

```json
{
  "type": "tint",
  "color": "#7A5CFF",
  "opacity": 0.14
}
```

Tint is composited over the permitted region. Renderers should clamp excessive opacity that would obscure native thumbnail content.

## Icon layer

Use the real app icon:

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
  "resource": "SpaceDress/alternate-icon.png",
  "placement": "topTrailing",
  "size": "large"
}
```

`bundleIcon` means the icon of the **actual runtime app bundle**. Readers should use macOS icon services rather than requiring authors to know whether the icon originated as `.icns` or a compiled asset catalog.

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

A renderer may replace `documentTitle` with `appTitle` or hide it when the user has disabled document-title display or Accessibility data is unavailable. Text presentation does not establish persistent document identity.

## Image layer

```json
{
  "type": "image",
  "resource": "SpaceDress/watermark.png",
  "placement": "fill",
  "contentMode": "fit",
  "opacity": 0.18,
  "blendMode": "sourceOver"
}
```

Resources are local to the approved app/style resource root. Supported v0.1 blend-mode names are `sourceOver`, `multiply`, `screen`, and `overlay`; a renderer may simplify unsupported modes with a diagnostic.

## Resource containment

Every app-manifest resource must be local and resolve under the app's `Contents/Resources` directory after canonicalization. Absolute paths, remote URLs, traversal outside the resource root, and resources that require helper execution are rejected.

Resources are decoded as bounded data, not treated as extensions to the application.

## Split/tiled full-screen semantics

An app manifest styles **its participant**, not the whole Space.

When participant geometry is known:

- tint/image/border layers are clipped to that participant region;
- icon/text placement is relative to that region;
- the app cannot cover the other participant.

When geometry is unknown but both applications are identified:

- readers/renderers should preserve both identities;
- full-region app tint/image layers may be suppressed;
- icon/text layers may be placed into a neutral paired layout;
- the renderer must not guess ownership geometry from unverified array order.

## User style packages

v0.1 standardizes the **visual manifest**, not the user's rule database.

SpaceDress may use the same manifest schema for user-authored styles while storing targeting/precedence separately. This keeps an app from declaring which other apps or documents it should control.

## Rendering safety

A conforming renderer may override requested presentation to protect usability:

- clamp opacity or border prominence;
- simplify crowded split layouts;
- hide document titles for privacy;
- respect Reduce Motion/Transparency and Increase Contrast;
- suppress a layer when its local resource fails validation;
- suppress all decoration when Mission Control frame alignment is unreliable.

The fallback for a bad manifest is a usable default, not a broken thumbnail.

## Extension policy

The root may contain an optional `extensions` object for namespaced experimental data. Core readers ignore unrecognized namespaces. Extensions do not bypass resource, privacy, or rendering-safety rules.

A broadly useful extension should graduate into a future version of the standard rather than become a private de facto API.

## Schema and examples

- [`spacedress-style.schema.json`](spacedress-style.schema.json)
- [`examples/minimal.json`](examples/minimal.json)
- [`examples/author-branded.json`](examples/author-branded.json)
- [`examples/split-friendly.json`](examples/split-friendly.json)

## Version history

See [`CHANGELOG.md`](CHANGELOG.md).
