# Product vision

## One sentence

**SpaceDress makes full-screen macOS Spaces visually distinguishable in Mission Control without replacing Mission Control or weakening macOS system integrity.**

## The original problem

Mission Control is visually efficient when applications have strongly different window appearances. It becomes much less efficient when several full-screen applications use dark themes, similar editor layouts, or otherwise low-distinction thumbnails.

The user already knows *what* is open. The expensive part is identifying *which thumbnail* corresponds to the desired full-screen context.

SpaceDress adds a compact identity layer to the existing thumbnail rather than asking the user to learn a replacement switcher.

## Primary user story

> When I open Mission Control, I can identify the full-screen application or split-screen pair I want with peripheral vision, without reading nearly identical window content.

That success criterion favors icons, color, and concise titles over rich miniature dashboards.

## Product hierarchy

### 1. Single-app full screen

This is the reference case. SpaceDress should determine the owning application, resolve its actual bundle, obtain a useful icon/title, resolve styling, and decorate the correct thumbnail.

### 2. Split/tiled full screen

A split Space is not "one Space with an awkward second window." It is a Space with multiple first-class participants. The identity model and rendering model must preserve that fact.

### 3. Regular Desktop Spaces

Regular desktops deserve reasonable support, but they are secondary. Their identity is inherently more ambiguous because many applications and windows may participate. SpaceDress may provide names or representative app sets without allowing those requirements to distort the full-screen design.

## Product principles

### Recognition before information density

A Space thumbnail should become easier to recognize at a glance. Decorations that require careful reading have failed the primary goal even if they contain more information.

### Preserve the native visual context

The thumbnail itself remains useful: it shows the actual application state. Decoration should frame or annotate it rather than cover it.

### Identity belongs to the workload, not the slot

"Desktop 4" or a native Space ID is not a durable user concept. A full-screen editor showing a project, a full-screen DAW session, or a split pair of apps is.

### Private integration is an adapter, not an architecture

SpaceDress will probably need undocumented macOS data because Apple does not expose the required Mission Control model as a supported public API. That fact must remain localized. The rest of the application should not know what an `SLSManagedSpace`, AX dictionary, or private plist looks like.

### No crowns required

In SpaceDress, a **title** means the textual name displayed for an app, document, or Space. It is a label—not an honorific, rank, ownership claim, or hereditary office.

### Local first

The problem is local to the Mac. SpaceDress should work without an account, cloud service, telemetry endpoint, or remote styling server.

### User intent outranks app preference

An application may ship a suggested SpaceDress style. The user always has the final word and can override or disable it.

## Non-goals

SpaceDress is not trying to:

- replace Mission Control;
- become a general tiling window manager;
- create, delete, move, or reorder Spaces;
- patch the Dock;
- require SIP changes;
- guarantee unsupported private APIs never break;
- execute application-supplied styling code;
- turn Mission Control thumbnails into interactive widgets;
- force regular Desktop Spaces into a false single-owner model.

## Design test

When evaluating a proposed feature, ask:

1. Does this make a full-screen Space easier to identify?
2. Can it work without weakening system integrity?
3. Does it keep unstable macOS knowledge behind an adapter?
4. Does it degrade safely when the platform refuses to cooperate?
5. Does it preserve the native thumbnail's usefulness?
6. Can the user override the app's preference?

A feature does not need six yes answers to be discussed, but a string of no answers is a strong sign it belongs in another project.
