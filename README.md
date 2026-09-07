# SpaceDress

> **Dapper attire for your Spaces.**

SpaceDress is a macOS utility and open styling standard for making **full-screen Spaces immediately recognizable in Mission Control**.

Modern apps often look nearly identical as Mission Control thumbnails—especially when everything is in dark mode. SpaceDress decorates those thumbnails with useful identity: application icons, titles, document titles, borders, tints, and optional artwork.

> [!IMPORTANT]
> SpaceDress is currently in the **design and feasibility** stage. This repository defines the product, architecture, integration policy, and style-manifest standard before implementation hardens those decisions into code.

## The problem

A row of dark full-screen windows is visually expensive to parse:

```text
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│              │ │              │ │              │
│ dark app UI  │ │ dark app UI  │ │ dark app UI  │
│              │ │              │ │              │
└──────────────┘ └──────────────┘ └──────────────┘
```

SpaceDress aims to turn that into something closer to:

```text
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│          ◉VS │ │          ◉RE │ │          ◉SF │
│              │ │              │ │              │
│ UHURA        │ │ AUDIO EDIT   │ │ RESEARCH     │
└──────────────┘ └──────────────┘ └──────────────┘
```

The native thumbnail remains native. SpaceDress supplies the identifying chrome.

## Full-screen first

SpaceDress is deliberately **not** a general virtual-desktop manager.

Priority order:

1. single-app full-screen Spaces;
2. split/tiled full-screen Spaces with two participants;
3. regular Desktop Spaces where support falls out naturally;
4. deeper Mission Control layout changes only if they can be achieved without compromising system integrity.

Changing the spacing between Apple's Space thumbnails would be attractive, but it is not a core goal. SpaceDress will not disable SIP or inject code into the Dock merely to gain layout control.

## What SpaceDress wants to show

A full-screen Space may be dressed with any combination of:

- the owning app's icon;
- application title;
- document/window title;
- border color and weight;
- translucent color overlay;
- bitmap artwork supplied by the app or user;
- split-screen participant identity;
- user overrides.

The renderer should make identification faster without obscuring the thumbnail it is identifying.

## Identity, not Space numbers

macOS exposes several identifiers for Spaces through undocumented or private interfaces. SpaceDress treats **all native Space identifiers as observations, not durable identity**.

A runtime Space may carry a managed-space ID, UUID-like value, display association, owner PIDs, window IDs, and Mission Control position. Those values are useful while they are true. None are allowed to become the durable key for user intent.

Persistent styling is instead resolved from logical identity such as:

- app bundle identifier;
- canonical app bundle location;
- code-signing identity when useful;
- runtime PID for same-bundle instance disambiguation;
- document identity when available and explicitly used;
- the ordered set of participants in a split full-screen Space.

See [The Space Model](docs/space-model.md).

## The SpaceDress Style Manifest

Apps should be able to ship their preferred Mission Control attire inside their own bundle. SpaceDress therefore defines a declarative, versioned manifest.

A minimal example:

```json
{
  "$schema": "https://raw.githubusercontent.com/kellyjanderson/spacedress/main/spec/spacedress-style.schema.json",
  "manifestVersion": "0.1",
  "layers": [
    {
      "type": "icon",
      "source": "bundleIcon",
      "placement": "topTrailing",
      "size": 40
    },
    {
      "type": "text",
      "source": "appTitle",
      "placement": "bottomLeading",
      "background": "material"
    }
  ]
}
```

The format is intentionally declarative: **no scripts, no remote assets, no arbitrary code execution**. See the [Style Manifest specification](spec/README.md).

## Architecture in one view

```text
macOS topology / private read SPI ─┐
Accessibility / Mission Control ───┼─> Space Snapshot
NSWorkspace / app bundles ─────────┘        │
                                            v
                                     Identity Resolver
                                            │
                user override ──────────────┼──── app manifest
                                            │
                                            v
                                       Style Resolver
                                            │
                                            v
                                      Overlay Renderer
```

Private macOS integration is isolated behind adapters. The domain model does not depend on SkyLight/CGS structures, native IDs, AX trees, or a particular macOS release.

## Project rules

- **No SIP weakening.** A feature that requires users to reduce macOS system integrity is outside the normal project boundary.
- **Read before write.** Private Space APIs may be explored for read-only discovery; undocumented mutation is treated much more skeptically.
- **Graceful degradation.** Missing private data removes a feature, not the app.
- **Local by default.** SpaceDress does not need telemetry, an account, or a network service to dress local Spaces.
- **Declarative styling.** App-provided styling is data, never executable code.
- **Fullscreen behavior defines success.** Regular desktops must not distort the design away from the problem SpaceDress exists to solve.

## Repository map

| Area | Purpose |
| --- | --- |
| [`docs/vision.md`](docs/vision.md) | Product intent, principles, and non-goals |
| [`docs/architecture.md`](docs/architecture.md) | Component boundaries and data flow |
| [`docs/space-model.md`](docs/space-model.md) | Runtime and persistent identity model |
| [`docs/style-guide.md`](docs/style-guide.md) | Visual and interaction dress code |
| [`docs/code-style.md`](docs/code-style.md) | Engineering conventions |
| [`docs/research/`](docs/research/) | macOS integration evidence and prior art |
| [`docs/decisions/`](docs/decisions/) | Architecture Decision Records |
| [`spec/`](spec/) | SpaceDress Style Manifest standard and schema |
| [`ROADMAP.md`](ROADMAP.md) | Sequenced feasibility and product milestones |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to contribute |

## Related work

SpaceDress builds on lessons demonstrated by projects such as Hammerspoon, AltTab, Rename Spaces, DesktopRenamer, SpaceNamer, CloseUp, Control Room, Rectangle, and other macOS utilities that work around the limits of Mission Control. They are research references, not dependencies unless explicitly listed later.

See [Prior Art](docs/research/prior-art.md).

## License and attribution

SpaceDress is licensed under the [Apache License 2.0](LICENSE).

Apache-2.0 permits commercial use and redistribution. It requires preservation of applicable copyright, license, and attribution notices and provides an express patent grant. The repository also includes a [`NOTICE`](NOTICE) file so attribution travels with derivative distributions.

The **SpaceDress** name and project identity are not granted as trademarks by the Apache license. See [`TRADEMARKS.md`](TRADEMARKS.md).

If SpaceDress is useful in research, software, or a derived project, [`CITATION.cff`](CITATION.cff) provides a machine-readable citation.

---

**SpaceDress** — the app's job is not to manage your Spaces. It is to make them easier to recognize.