# Prior art and research references

SpaceDress should learn from existing macOS utilities without inheriting their assumptions blindly. This list records what each project is useful for investigating.

No project listed here is a SpaceDress dependency merely because it is a reference.

## Hammerspoon — `hs.spaces`

Repository: <https://github.com/Hammerspoon/hammerspoon>

Why it matters:

- mature use of private Space APIs;
- managed-display Space enumeration;
- Mission Control Accessibility-tree inspection;
- examples of version-sensitive Space behavior.

Research question: which data paths remain reliable on the oldest/current macOS releases SpaceDress intends to support?

## NUIKit / CGSInternal

Repository: <https://github.com/NUIKit/CGSInternal>

Why it matters:

- historical declarations for CGS Space functions;
- `CGSSpaceCopyOwners` documents an owner-PID concept directly relevant to full-screen Spaces;
- useful vocabulary/signature history for private adapter research.

Treat headers as reverse-engineered historical evidence, not Apple documentation.

## AltTab

Repository: <https://github.com/lwouis/alt-tab-macos>

Why it matters:

- large, mature Swift macOS utility operating across Spaces/windows;
- practical Accessibility and private integration lessons;
- compatibility history across many macOS releases.

## Rename Spaces

Repository: <https://github.com/egwoo/rename-spaces>

Why it matters:

- directly overlays names in Mission Control;
- works without modifying native Space names;
- documents that Mission Control AX elements may be incomplete and geometric fallback may be necessary.

This is a particularly direct proof that SpaceDress's overlay strategy is worth pursuing.

## DesktopRenamer

Repository: <https://github.com/gitmichaelqiu/DesktopRenamer>

Why it matters:

- modern Space naming and Space API work;
- runs without disabling SIP;
- useful reference for persistence and multi-Space UX.

## SpaceNamer

Repository: <https://github.com/brandaorafael/SpaceNamer>

Why it matters:

- reads managed Space data through private SkyLight with SIP enabled;
- uses native Space UUIDs successfully for its own desktop-name persistence;
- automatically names full-screen Space state from the frontmost app.

SpaceDress deliberately chooses a stricter identity rule: even if UUID persistence works in SpaceNamer's supported environment, native identifiers remain hints rather than the durable key for full-screen styling.

## Control Room

Repository: <https://github.com/dev-jonghoonpark/control-room>

Why it matters:

- recent macOS project using private SkyLight to list Spaces/windows and capture other-Space previews;
- clear example of direct distribution/notarization tradeoffs caused by private APIs.

## CloseUp

Repository: <https://github.com/oomol-lab/CloseUp>

Why it matters:

- augments native Mission Control with overlay controls;
- demonstrates a much richer overlay interaction than SpaceDress initially requires.

If interactive overlays are possible, SpaceDress's noninteractive icon/title/border overlay should be a lower-complexity target—though geometry and lifecycle still require independent testing.

## Lattice

Repository: <https://github.com/bryancostanich/lattice>

Why it matters:

- modern experiments with Space navigation, per-Space anchors, and thumbnails;
- documents behavior of separate-Spaces and automatic Space rearrangement settings;
- keeps SIP enabled while accepting private API limitations.

## Rectangle

Repository: <https://github.com/rxhanson/Rectangle>

Why it matters to repository design more than Space discovery:

- mature, popular native macOS utility;
- relatively simple contribution surface;
- good model for keeping a focused utility understandable without enterprise process overhead.

## Homebrew and Swift project repositories

Repositories:

- <https://github.com/Homebrew/brew>
- <https://github.com/swiftlang/swift>
- <https://github.com/swiftlang/repo-templates>

Why they matter:

- community-health conventions;
- explicit contribution and security guidance;
- repository templates, CODEOWNERS, issue/PR structure;
- emphasis on reviewable evidence and maintainable project process.

SpaceDress borrows the *discipline* of these repositories, not their scale or bureaucracy.

## Updating this document

When a referenced project materially changes its strategy, record the observation date/macOS version rather than silently treating a current README as timeless evidence.
