# Code style and engineering guide

This guide establishes design conventions before the implementation exists. Update it when real code demonstrates a better pattern; do not preserve a rule merely because it was written first.

## Language and platform

The primary application is expected to be written in Swift using AppKit/SwiftUI and Apple system frameworks where they fit.

Do not select a minimum macOS version or Swift language mode merely for aesthetic modernity. The supported matrix should follow the oldest release on which the required Mission Control integration can be maintained reliably.

## Formatting

- 4-space indentation for Swift.
- UTF-8, LF line endings, final newline.
- Prefer the official `swift-format` ecosystem once source exists and pin its configuration in the repository.
- Avoid formatting-only churn mixed into semantic changes.

## Naming

- Types and protocols: `UpperCamelCase`.
- Functions, variables, cases: `lowerCamelCase`.
- Use domain terms from `docs/space-model.md` consistently.
- Include units in ambiguous numeric names: `timeoutMilliseconds`, `opacity`, `thumbnailWidthPoints`.
- Avoid unexplained abbreviations except established platform terms (`PID`, `AX`, `SLS`, `CGS`, `URL`).

### Private API names

When wrapping a private symbol, preserve Apple's symbol name only at the foreign-function boundary. Immediately translate to a project-owned method/type.

Good:

```text
SkyLightSpaceTopologyProvider.spaces(for: display)
```

Avoid leaking:

```text
SLSCopyManagedDisplaySpacesResultDictionary
```

through the application.

## Module boundaries

Prefer modules/targets organized by dependency rather than feature-screen folder alone:

```text
SpaceDressDomain
SpaceDressPlatform
SpaceDressStyle
SpaceDressUI
SpaceDressApp
```

The exact target layout can evolve, but domain logic must remain testable without loading private frameworks.

## Private macOS SPI

All private symbols must be:

- declared/resolved in one platform layer;
- documented with observed signatures and macOS versions tested;
- availability-checked at runtime;
- guarded against `nil`/shape changes;
- translated into domain models before leaving the adapter;
- covered by a capability/failure path.

Prefer dynamic symbol resolution where it meaningfully reduces hard linking to private frameworks, but do not confuse `dlsym` with a stability guarantee.

Do not scatter magic dictionary keys through the codebase. Parse private structures in one place and add fixtures from sanitized observations when legally and technically appropriate.

## Error handling

Platform uncertainty is ordinary control flow.

Use typed errors/capability results that distinguish:

- unsupported OS/interface;
- permission denied;
- incomplete data;
- malformed private data;
- transient Mission Control state;
- internal invariant violation.

Do not use `fatalError`, forced casts, or force unwraps for private API results in shipping paths.

## Concurrency

- UI and `NSWindow`/AppKit state belong on `@MainActor`.
- Platform probing that can block should not run synchronously on the main actor.
- Prefer structured concurrency and explicit task ownership.
- Cancellation must be honored when Mission Control closes or a newer snapshot supersedes an older one.
- Avoid detached tasks unless isolation and lifetime are intentionally independent.

## State

Prefer immutable snapshots passed through the pipeline over a global mutable model continuously edited by adapters.

A useful pattern:

```text
Platform observations → immutable SpaceSnapshot → resolved style → render plan
```

The next observation replaces or reconciles the previous snapshot in one coordinator.

## Logging

Use `os.Logger`/unified logging once implementation begins.

Log categories should follow stable subsystems such as:

- `topology`
- `ownership`
- `mission-control`
- `identity`
- `manifest`
- `rendering`

Privacy rules:

- PID and bundle identifier are generally acceptable diagnostics;
- document titles, file names, full paths, screenshots, and window content are private by default;
- avoid normal-level logging of volatile AX values;
- provide explicit diagnostic export/redaction rather than asking users to scrape Console blindly.

## Dependencies

Every third-party dependency adds release, security, and compatibility cost.

Before adding one, document:

- what capability it supplies;
- why Apple/Swift standard libraries are insufficient;
- binary/source size impact where material;
- license compatibility;
- maintenance activity;
- whether the dependency reaches private APIs itself.

Small platform shims are often safer to own than importing an entire window-management framework.

## Tests

### Domain tests

Pure domain logic should have deterministic unit tests for:

- identity matching;
- rule precedence;
- split composition;
- manifest validation/normalization;
- reconciliation;
- graceful fallback.

### Platform contract tests

Private integrations need versioned contract tests/probes that capture:

- symbol availability;
- returned type/shape;
- expected fields;
- behavior on current vs non-current Spaces;
- single vs split full-screen behavior;
- multi-display variations.

A contract test failing on a new macOS beta is information, not merely a red CI light.

### UI tests

Visual overlay tests should verify geometry and lifecycle rather than pixel-perfect private Mission Control rendering.

## Comments

Comments should explain:

- why a workaround exists;
- what macOS behavior was observed;
- which invariant would be violated without the code;
- what would permit removing the workaround.

Do not narrate obvious Swift syntax.

## Source headers

New original source files should carry a compact SPDX header where practical:

```text
Copyright 2026 Kelly Eldritch and SpaceDress contributors
SPDX-License-Identifier: Apache-2.0
```

Do not add contributor-by-contributor copyright banners to files.
