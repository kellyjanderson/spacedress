# SpaceDress

> **Dapper attire for your Spaces.**

SpaceDress is a macOS utility for making **full-screen Spaces immediately recognizable in Mission Control**.

Modern applications often look nearly identical as Mission Control thumbnails—especially when everything is in dark mode. SpaceDress adds useful identity to those thumbnails: application icons, titles, document titles, borders, tints, and optional artwork.

SpaceDress is also an implementation and early steward of the **Desktop Switcher Appearance Specification (DSAS)**, a neutral application-facing format that is deliberately not named after SpaceDress.

> [!IMPORTANT]
> SpaceDress is currently in the **design and feasibility** stage. This repository defines the product, architecture, integration policy, user-configuration model, and the experimental DSAS before implementation hardens those decisions into code.

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

Changing the spacing between Apple's Space thumbnails would be useful, but it is not a core goal. SpaceDress will not disable SIP or inject code into the Dock merely to gain layout control.

## Where appearance comes from

SpaceDress has two configuration authorities.

### 1. The application

An application should look useful in SpaceDress **without knowing SpaceDress exists**.

SpaceDress first derives a baseline appearance from the actual running application:

- bundle icon;
- localized application name;
- document/window title when available and permitted;
- other bounded metadata that can be obtained safely from the running app and its bundle;
- optional derived palette information, such as an accent inferred from the application icon.

If an application someday ships a **Desktop Switcher Appearance Manifest (DSAM)**, that manifest can explicitly refine the application-side appearance. DSAM is defined by DSAS and is not a SpaceDress-branded format.

No manifest is the normal case. Missing DSAM data is not an error and must never make an app less identifiable.

### 2. The user

SpaceDress maintains its own **multi-app user manifest** for customization and rules. It can target many applications and override application-derived or application-declared values.

That multi-app manifest is internal SpaceDress machinery. It is **not part of DSAS**, is not an application integration surface, and carries no promise that another DSAS renderer will understand it.

Resolution is conceptually:

```text
actual running app
      │
      ├─ derive baseline appearance
      │
      └─ overlay optional DSAM declarations
                    │
                    v
          application appearance
                    │
                    v
        SpaceDress user overrides
                    │
                    v
             resolved appearance
```

The user has final authority.

## Identity, not Space numbers

macOS exposes several identifiers for Spaces through undocumented or private interfaces. SpaceDress treats **all native Space identifiers as observations, not durable identity**.

A runtime Space may carry a managed-space ID, UUID-like value, display association, owner PIDs, window IDs, and Mission Control position. Those values are useful while they are true. None are allowed to become the durable key for user intent.

Persistent styling is instead resolved from logical identity such as:

- app bundle identifier;
- canonical app bundle location;
- code-signing identity when useful;
- runtime PID for same-bundle instance disambiguation;
- document identity when available and explicitly used;
- the participant set in a split full-screen Space.

See [The Space Model](docs/space-model.md).

## Split/tiled full-screen geometry

Split full-screen participants are not merely an unordered pair of applications.

SpaceDress aims to resolve the actual windows belonging to the Space and retain their geometry. From those bounds it can derive:

- physical left/right placement;
- participant width and split ratio;
- normalized participant rectangles;
- future top/bottom or more complex arrangements if macOS exposes them.

Owner-array order is never treated as geometry.

Conceptually:

```text
Space
└── participants[]
    ├── app identity
    ├── window identity
    ├── screen-space bounds
    ├── normalized region
    └── derived physical side
```

Mission Control decoration can then transform each normalized participant region into the corresponding region of the Space thumbnail. A 70/30 split remains a 70/30 split rather than becoming two guessed halves.

## Desktop Switcher Appearance Specification

The public standard is the **Desktop Switcher Appearance Specification (DSAS)**.

A conforming application document is a **Desktop Switcher Appearance Manifest (DSAM)**.

The naming is intentionally descriptive and implementation-neutral. SpaceDress is one renderer; the specification should remain useful to another macOS utility—or eventually another desktop environment—without adopting SpaceDress branding.

A minimal DSAM example:

```json
{
  "$schema": "https://raw.githubusercontent.com/kellyjanderson/spacedress/main/spec/desktop-switcher-appearance.schema.json",
  "manifestVersion": "0.1",
  "layers": [
    {
      "type": "border",
      "color": "#7A5CFF",
      "width": "medium"
    },
    {
      "type": "icon",
      "source": "bundleIcon",
      "placement": "topTrailing",
      "size": "large"
    }
  ]
}
```

The format is intentionally declarative: **no scripts, no remote assets, no arbitrary code execution**. See the [Desktop Switcher Appearance Specification](spec/README.md).

## Architecture in one view

```text
macOS topology / private read SPI ─┐
Accessibility / Mission Control ───┼─> Space Snapshot
NSWorkspace / app bundles ─────────┘        │
                                            v
                                     Identity Resolver
                                            │
                         ┌──────────────────┴─────────────────┐
                         v                                    v
                Application Appearance               SpaceDress User Manifest
                derived + optional DSAM                       │
                         └──────────────────┬─────────────────┘
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
- **Useful without adoption.** Applications require no manifest or integration for SpaceDress to identify them.
- **Neutral public standard.** DSAS is not a SpaceDress configuration format.
- **User configuration stays private to SpaceDress.** The multi-app user manifest is implementation machinery, not part of DSAS.
- **Geometry beats ordering.** Split placement comes from window geometry, never owner-array position.
- **Fullscreen behavior defines success.** Regular desktops must not distort the design away from the problem SpaceDress exists to solve.

## Repository map

| Area | Purpose |
| --- | --- |
| [`docs/vision.md`](docs/vision.md) | Product intent, principles, and non-goals |
| [`docs/architecture.md`](docs/architecture.md) | Component boundaries and data flow |
| [`docs/space-model.md`](docs/space-model.md) | Runtime and persistent identity model |
| [`docs/user-configuration-model.md`](docs/user-configuration-model.md) | Internal multi-app customization model |
| [`docs/style-guide.md`](docs/style-guide.md) | Visual and interaction dress code |
| [`docs/code-style.md`](docs/code-style.md) | Engineering conventions |
| [`docs/research/`](docs/research/) | macOS integration evidence and prior art |
| [`docs/decisions/`](docs/decisions/) | Architecture Decision Records |
| [`spec/`](spec/) | Desktop Switcher Appearance Specification and schema |
| [`ROADMAP.md`](ROADMAP.md) | Sequenced feasibility and product milestones |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to contribute |

## Related work

SpaceDress builds on lessons demonstrated by projects such as Hammerspoon, AltTab, Rename Spaces, DesktopRenamer, SpaceNamer, CloseUp, Control Room, Rectangle, yabai, and other macOS utilities that work around the limits of Mission Control. They are research references, not dependencies unless explicitly listed later.

See [Prior Art](docs/research/prior-art.md).

## License and attribution

SpaceDress is licensed under the [Apache License 2.0](LICENSE).

Apache-2.0 permits commercial use and redistribution. It requires preservation of applicable copyright, license, and attribution notices and provides an express patent grant. The repository also includes a [`NOTICE`](NOTICE) file so attribution travels with derivative distributions.

The **SpaceDress** name and project identity are not granted as trademarks by the Apache license. DSAS deliberately uses a descriptive name rather than depending on SpaceDress branding. See [`TRADEMARKS.md`](TRADEMARKS.md).

If SpaceDress is useful in research, software, or a derived project, [`CITATION.cff`](CITATION.cff) provides a machine-readable citation.

---

**SpaceDress** — the app's job is not to manage your Spaces. It is to make them easier to recognize.
